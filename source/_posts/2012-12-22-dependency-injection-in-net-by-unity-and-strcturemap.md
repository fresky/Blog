---
layout: post
title: ".NET中使用Unity和StructureMap来实现依赖注入Dependency Injection"
date: 2012-12-22
comments: true
tags: CSharp
---
<p>本文用一个非常简单的示例来演示一下如何使用<a href="http://unity.codeplex.com/">Unity</a>和<a href="http://docs.structuremap.net/index.html">StructureMap</a>在C#中实现Dependency Injection。</p>  

<p>我们来做一个非常简单的程序，这个程序会把用户输入的字符串做个逆序，然后输出，同时要求记录一下每次用户的输入和结果，我们支持两种Logger，一种是命令行的，一种是对话框的，用户可以选择使用哪种Logger。</p>  <p>界面如下：</p>  

{% limg DIExample.png %}

这个程序使用MVP来实现的，我们有4个接口如下，分别对应V，P，M和Logger：


```csharp
    public interface IView
    {
        void DisplayResult(string result);
    }
    public interface IPresenter
    {
        void HandleReverse(string text, IView view);
        void SetLogger(ILogger logger);
    }
    internal interface IModel
    {
        string Reverse(string text);
    }
    public interface ILogger
    {
        void LogMessage(string message);
    }
```

<h3>1. 手工实现依赖注入和singleton。</h3>

<p>在App.xaml.cs中我们通过下面的方法来打开UI。可以看到我们用了一堆的new关联起来，创建了一个view。</p>

```
	protected override void OnStartup(StartupEventArgs e)
	{
		createViewByHand().Show();
	}
	private static Window createViewByHand()
	{
		return new MainWindow(new Presenter(LoggerFactory.CreateLogger(), ModelFactory.CreateModel()));
	}
```

<p>这里ModelFactory和LoggerFactory是为了保证我们的应用中只有一个Model和Logger的实例。（其实主要是logger，因为model在我们的应用里就一个，但是用户可以选择不同的logger，我们不希望每回用户选择之后都生成一个新的logger）</p>

<p>用户选择下拉框的代码如下：</p>

```
	private void loggerTypeChanged(object sender, SelectionChangedEventArgs e)
	{
		m_Controller.SetLogger(getLoggerByHand());
	}
	private ILogger getLoggerByHand()
	{
		return LoggerFactory.CreateLogger(loggerType.SelectedItem.ToString());
	}
```

<h3>2. 使用StructureMap实现依赖注入和singleton。</h3>

<p>代码如下，我们创建一个Container，配置对应于每个接口，我们使用哪个实例。注意我们在ILogger和IModel中使用了Singleton方法，同时我们配置了2个ILogger的实例，分别设置了一个名字，在用户改变下拉框时，我们从Container中取出相应的实例。</p>

```
	private static Window createViewByStructureMap()
	{
		StructureMapContainer = new StructureMap.Container(x =>
		{
			x.For<IPresenter>().Use<Presenter>();
			x.For<IView>().Use<MainWindow>();
			x.For<IModel>().Singleton().Use<ReverseModel>();
			x.AddType(typeof(ILogger), typeof(MessageBoxLogger), LoggerFactory.MessageboxLoggerName);
			x.AddType(typeof(ILogger), typeof(ConsoleLogger), LoggerFactory.ConsoleLoggerName);
			x.For<ILogger>().Singleton().UseSpecial(y => y.TheInstanceNamed(LoggerFactory.ConsoleLoggerName));
		});
		StructureMapContainer.AssertConfigurationIsValid();

		return StructureMapContainer.GetInstance<MainWindow>();
	}

	private ILogger getLoggerByStructureMap()
	{
		return App.StructureMapContainer.GetInstance<ILogger>(loggerType.SelectedItem.ToString());
	}
```

<h3>3. 使用Unity实现依赖注入和singleton。</h3>

<p>代码非常类似，只是用ContainerControlledLifetimeManager来实现singleton。</p>

```
	private Window createViewByUnity()
	{
		UnityContainer = new UnityContainer();
		UnityContainer.RegisterType<IPresenter, Presenter>();
		UnityContainer.RegisterType<IView, MainWindow>();
		UnityContainer.RegisterType<IModel, ReverseModel>(new ContainerControlledLifetimeManager());
		UnityContainer.RegisterType<ILogger, ConsoleLogger>(LoggerFactory.ConsoleLoggerName,
															new ContainerControlledLifetimeManager());
		UnityContainer.RegisterType<ILogger, MessageBoxLogger>(LoggerFactory.MessageboxLoggerName,
															   new ContainerControlledLifetimeManager());
		return UnityContainer.RegisterInstance(typeof(ILogger), UnityContainer.Resolve<ILogger>(LoggerFactory.ConsoleLoggerName)).Resolve<MainWindow>();
	}

	private ILogger getLoggerByUnity()
	{
		return App.UnityContainer.Resolve<ILogger>(loggerType.SelectedItem.ToString());
	}
```

<p>另外StructureMap和Unity都支持Attribute来制定应该往哪个属性上Inject。StructureMap是[SetterProperty]，Unity是 [Dependency]。</p>

<p>具体的实现就不详细列出来，可以在<a href="https://github.com/fresky/DIExample">github</a>上找到源码。

  

  

  

  

  
</p>