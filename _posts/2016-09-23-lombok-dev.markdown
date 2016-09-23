---
layout: post
title:  Lombok扩展，打开Java的另一扇门
tags: Java Annotation Lombok
categories: Java
---
Java自从1.5开始引入了Annotation，这赋予了Java语言新的活力，现在很多框架或者技术都或多或少地使用了Annotation的功能，逐渐成为一种新的趋势。

某种程度上，其中最值得关注的当属[Lombok](https://projectlombok.org/)了, 从编码角度精简了程序，消除了许多无聊的代码（比如@Setter会自动生成Field的setter方法）；另一方面，最近在研究Mock数据的主题，以Annotation的方式实现是比较容易想到的途径之一, 这样可以最小化最开发人员的影响，Mock通过配置文件的方式来实现，只需要在需要Mock的地方打个Annotation就足够了。

先看看Javac的处理流程
![编译流程](/images/javac-flow.png)

原则上我们不能修改源代码Parse后的AST（抽象语法树），但是Javac可以使用未公开的API来实现这个步骤，许多IDE也都是利用了这一点，比如Eclipse，下面是它检查到错误，给出错误提示，然后根据用户选择的解决方法后修改AST，然后再同步成源代码的过程

![Eclipse AST Change](/images/eclipse-workflow.png)

Lombok封装了Annotation处理的过程，让我们可以快速方便直观地修改AST，以我们Mock为例:

第一步: 定义Annotation

{% highlight java %}
@Target({ElementType.LOCAL_VARIABLE, ElementType.FIELD})
public @interface Mock {
    String value() default "";
}
{% endhighlight %}

第二步: 扩展Lombok的JavacAnnotationHandler

{% highlight java %}
@MetaInfServices(JavacAnnotationHandler.class)
public class HandleMock extends JavacAnnotationHandler<Mock> {

    private static Configration conf = Configration.get();

    @Override
    public void handle(AnnotationValues<Mock> annotation, JCAnnotation ast, JavacNode annotationNode) {

        deleteAnnotationIfNeccessary(annotationNode, Mock.class);

        if( conf == null ) {
            annotationNode.addError("Cannot load mock.json profile.");
            return;
        }

//        annotationNode.addError(conf.toString());

        if( !conf.isEnabled() ) {
            // do nothing since mock is OFF
            return;
        }

        if (annotationNode.up().getKind() != Kind.LOCAL) {
            annotationNode.addError("@Mock is legal only on local variable declarations.");
            return;
        }

        JCVariableDecl decl = (JCVariableDecl)annotationNode.up().get();

        if (decl.init == null) {
            annotationNode.addError("@Mock variable declarations need to be initialized.");
            return;
        }

        String var = decl.name.toString();

        if ( !conf.exist(var) ) {
            annotationNode.addError( var + " is need to mock, but not in mock conf");
            return;
        }

        JavacNode ancestor = annotationNode.up().directUp();
        JCTree blockNode = ancestor.get();

        final List<JCStatement> statements;
        if (blockNode instanceof JCMethodDecl) {
            statements = ((JCMethodDecl)blockNode).body.stats;
        } else {
            annotationNode.addError("@Mock is legal only on a local variable declaration inside a block.");
            return;
        }

        ListBuffer<JCStatement> newStatements = new ListBuffer<JCStatement>();
        for (JCStatement statement : statements) {
            if ( statement != decl ) {
                newStatements.append(statement);
            } else {
                newStatements.appendList(applyChange(annotationNode, statement, var));
            }
        }

        if (blockNode instanceof JCMethodDecl) {
            ((JCMethodDecl)blockNode).body.stats = newStatements.toList();
        } else throw new AssertionError("Should not get here");

        ancestor.rebuild();
    }

    private List<JCStatement> applyChange(JavacNode node, JCStatement stat, String var) {
        JavacTreeMaker maker = node.getTreeMaker();
        JCVariableDecl decl = (JCVariableDecl)stat;

        ListBuffer<JCStatement> newStatements = new ListBuffer<JCStatement>();

        JCVariableDecl mockedVarDecl = HandleMock.createMockedVar(node, maker, decl);
        newStatements.append(mockedVarDecl);

        for( ConfItemProp cip: conf.queryProps(var) ) {
            JCFieldAccess setProperty = maker.Select(maker.Ident(mockedVarDecl.name), node.toName(getSetterName(cip.prop)));
            ListBuffer<JCExpression> args = new ListBuffer<JCExpression>();
            JCLiteral literal = maker.Literal(cip.value);
            args.append(literal);

            List<JCStatement> setPropertyCall = List.<JCStatement>of(maker.Exec(
                    maker.Apply(List.<JCExpression>nil(), setProperty, args.toList())));

            newStatements.appendList(setPropertyCall);
        }

        return newStatements.toList();
    }

    private static JCVariableDecl createMockedVar(final JavacNode node,
                                                         final JavacTreeMaker maker, final JCVariableDecl decl) {
        JCExpression newClassExpr = maker.NewClass(null, List.<JCExpression>nil(),
                decl.vartype,
                List.<JCExpression>nil(),
                null);

        return maker.VarDef(maker.Modifiers(FINAL), decl.name, decl.vartype, newClassExpr);
    }

    private static String generateVarName() {
        int rand = (int)(Math.random()*100);
        long ts = System.currentTimeMillis();
        return "____mocked_var_" + ts + "_" + rand + "____";
    }

    private static String getSetterName(String var) {
        return "set" + var.substring(0, 1).toUpperCase() + var.substring(1);
    }

}
{% endhighlight %}

需要注意一下几点:

* org.kohsuke.MetaInfServices是为了生成META-INF/services下对应的服务配置文件，原始的是`javax.annotation.processing.Processor`, Lombok封装了的是`lombok.javac.JavacAnnotationHandler`
* Annotation处理方式是找到上一层代码， 遍历每个Child加入新的子树，遍历到当前节点时，根据当前节点，生成变换后的子树， 最后统一将父层的body替换
* 使用了set注入属性的方式，如果继续做，会考虑和spring一样的配置支持（比如构造注入等）以及支持非String类型的配置

配置文件如下:

{% highlight javascript %}
{
  "enabled": 0,
  "mocks": [
    {
      "var": "person",
      "props": [
        {"prop":"name", "value": "Mocked!"},
        {"prop":"intro", "value": "I am mocked Person!"}
      ]
    }, {
      "var": "car",
      "props": [
        {"prop":"brand", "value": "XXX"},
        {"prop":"model", "value": "C200"}
      ]
    }
  ]
}
{% endhighlight %}

第三步: 测试代码

{% highlight java %}
public class Car {
    String brand;
    String model;

    public Car() {
    }

    public Car(String brand, String model) {
        this.brand = brand;
        this.model = model;
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public String getModel() {
        return model;
    }

    public void setModel(String model) {
        this.model = model;
    }

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", model='" + model + '\'' +
                '}';
    }
}
public class TestMock {
    public void testMock() {
        @Mock Car car = new Car("BMW", "X6");
        System.out.println(car);
    }
}
{% endhighlight %}

编译后的代码

{% highlight java %}
public class TestMock {
    public TestMock() {
    }

    public void testMock() {
        System.out.println(person);
        Car car = new Car();
        car.setBrand("XXX");
        car.setModel("C200");
        System.out.println(car);
    }
}
{% endhighlight %}
