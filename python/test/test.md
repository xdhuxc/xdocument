### unittest
#### 核心概念
* TestCase：测试用例，所有测试用例的基类，是软件测试中最基本的组成单元。一个 TestCase 对象就是一个测试用例，是一个完整的测试流程，包括测试前环境的搭建 setUp，执行测试代码 run，以及测试后环境的还原 tearDown。每一个测试用例是一个完整的测试单元，可以对某一问题进行验证。
* TestSuite：测试套件，多个测试用例的集合就是测试套件，TestSuite 可以嵌套 TestSuite。
* TestLoader：是用来加载 TestCase 到 TestSuite 中的，其中有几个 loadTestFrom_() 方法，就是从各个地方寻找 TestCase，创建它们的实例，然后添加到 TestSuite 中，再返回一个 TestSuite 实例。
* TextTestRunner：是来执行测试用例的，其中的 run(test) 会执行 TestSuite/TestCase 中的 run(result) 方法。
* TextTestResult：测试结果会保存到 TextTestResult 实例中，包括运行了多少测试用例，成功与失败多少等信息。
* TestFixture：测试脚手，测试代码的运行环境，指测试准备前和执行后要做的工作，包括 setUp 和 tearDown 方法。

#### 测试流程
1、写好 TestCase，一个类继承 unittest.TestCase，就是一个测试用例，其中有多个以 test 开头的方法，那么每一个这样的方法，在加载的时候，会生成一个 TestCase 实例。如果一个类中有四个 test 开头的方法，最后加载到测试套件中时则有四个测试用例。

2、由 TestLoader 加载 TestCase 到 TestSuite。

3、然后由 TextTestRunner 来运行 TestSuite，运行的结果保存在 TextTestResult 中。

说明：
a）通过命令行或者 unittest.main() 执行时，main 会调用 TextTestRunner 中的 run 来执行，或者可以直接通过 TextTestRunner 来执行用例。

b）Runner 执行时，默认将结果输出到控制台，可以设置其输出到文件，在文件中查看结果，也可以通过 HTMLTestRunner 将结果输出到 HTML。

#### unittest 实例
##### 测试代码

##### 运行结果




说明：
a）在第一行给出了每一个用例执行的结果标识，成功为：.，失败为：F，出错为：E，跳过为：S。另外注意，测试的执行跟方法的顺序没有关系。

b）每个测试方法均以 test 开头，否则是不被 unittest 识别的。

c）在 unittest.main() 中加 verbosity 参数可以控制输出的错误报告的详细程度，默认是：1，如果设为：0，则不输出每一用例的执行结果，即没有上面的结果中的第一行；如果设为：2，则输出详细的执行结果。

##### 组织 TestSuite
a）确定测试用例的顺序，哪个先执行，哪个后执行？

b）如果测试文件有多个，怎么进行组织？

TestLoader 加载 TestCase 的几种方法：






