### unittest
#### 核心概念
* TestCase：测试用例，所有测试用例的基类，是软件测试中最基本的组成单元。一个 TestCase 对象就是一个测试用例，是一个完整的测试流程，包括测试前环境的搭建 setUp，执行测试代码 run，以及测试后环境的还原 tearDown。每一个测试用例是一个完整的测试单元，可以对某一问题进行验证。
* TestSuite：测试套件，多个测试用例的集合就是测试套件，TestSuite 可以嵌套 TestSuite。
* TestLoader：是用来加载 TestCase 到 TestSuite 中的，其中有几个 loadTestFrom_() 方法，就是从各个地方寻找 TestCase，创建它们的实例，然后添加到 TestSuite 中，再返回一个 TestSuite 实例。
* TextTestRunner：是来执行测试用例的，其中的 run(test)会执行 TestSuite/TestCase 
* TextTestResult：
* TestFixture：

#### 测试流程
