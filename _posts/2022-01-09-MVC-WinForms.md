Old apps are often useful but difficult to replace. Often they become this huge mess of tangled code that no one can fix without fear of introducing more bugs. You will hear programmers say, "This app needs to be re-written". Well rewriting is one way, here I show you another.

The **MVC** I show you here is the easiest to follow if you are new to this. It allows you split the code out of a WinForms app with minimal impact. You can do one small part at a time. You may want to implement the full MVC pattern, then simply continue from where this leaves off. My plan is to make the code easier to maintain and even replace the UI to something more modern.

**WHY?** The reason to refactor becomes apparent when you look at an old WinForms app. Ok some are beautiful, but its rare.

This small project [interval refactor project](https://github.com/richard-porteous/interval-refactor-project ".net C# WinForms MVC style") takes a WinForms C# app called Interval and adds MVC design to it. The app has a very simple UI. It doesn't use a database, so you wont see much in the model.

1. The project start the 2nd commit (step 0) with IntervalView which is the code we will modify.
2. The next commit adds the IntervalModel and IntervalController

The objective here is to separate UI code from logic and from database queries.

1. WinForms is the UI
2. controller class
3. model class.
4. Interface C# class

*UI Class. (WinForms)* View or display. To keep this simple we let the forms load as normal. On an existing app this can be changed once your code is cleaner. The code behind deals with UI related things, and it makes calls to the controller when something on the UI is changed by the user.

*Controller Class.* Called by UI and Calls the UI. Runs logic, calculations and to update the views with data. Often in these changes you will see that these apps are UI and Database with very little between. Its our job to separate the fact from the fiction. As you will see in later blogs.

*Model Class.* Database code not much else.

*Interface.* C# class allowing us to reduce the dependency between View and controller. It will contain one abstract method called update() which is what the controller can call to tell the WinForms code that it can reload the data into the fields. NOTE: I added this last as I was dealing with one Winform class. I definitely suggest adding it before the second WinForm.

This project simply shows you how to add the controller (and its interface) and model, nothing more. See the project readme for more explanation. This can be added without changing any other code and without affecting any behavior. The refactoring of the code is dealt with later.

If you have an existing project all you will need to do to start is create the controller and model and for each WinForm and an Interface for the controller. When a WinForm is there to support it *could possibly* use the same controller and model - i.e. "Customers" and "CustomerAdd" are two different WinForms with much in common. Here are a few snippets from the project.

In the UI - we add and initialize the controller.
```
public partial class IntervalView : Form
{
    private IntervalController controller;
    public IntervalView()
    {
        InitializeComponent();

        IntervalController ctl = new IntervalController(this);
        SetController(ctl);
    }

    internal void SetController(IntervalController cont)
    {
        controller = cont;
        Update();
    }

    public void Update() {

    }
  }

```

In the Controller - we add and initialize the model, and use the passed view. We could use an object for the view or an Interface. In a later commit I make an 'interface' and that exposes the WinForms method 'update' and nothing else.

```
class IntervalController
{
    private IntervalModel model;
    private IntervalView view;

    public IntervalController(IntervalView view)
    {
        this.model = new IntervalModel();
        this.view = view;
    }

    private void SetChanged() { }

    private void NotifyObservers()
    {
      view.Update();
    }
}
```

And in the Model - we don't need to do anything, just yet. I would assume you can set up a connection to the database and would have the calls to the database in their separate functions.

```
class IntervalModel
{
    public IntervalModel()
    {
    }
}
```

View the repository to see the example code in action. There is only one branch so you will need to be able to see each commit.
For tools: I used visual studio community, C#, and .NET 6. Modify as you please.

The example app shows one way how you could implement MVC. You may find following the pattern closely makes for easier to read code. Investigate he MVC model further, it could transform your style.

*NOTE*
WinForms for .NET 5 & 6 no longer supports the SQL reporting.
But I suggest you have your reports in a separate project as it helps keep the code separate and allows for various options.

If you want to keep your older reports, you could try [FASTREPORT](https://fastreports.github.io/FastReport.Documentation/ "Fast-Report") which has a community version and is available via NuGet. Not all reports convert well and there may be need for some manual adjustments.

That's it for today. If you see any bugs in the repository feel free to drop me a note. The code is free to use, the repository can be forked or downloaded.


###General

When you make these changes take small steps.
Separate the code in the project one variable at a time.
Move the code into a function that needs to be moved to controller or model. Change Globals into parameters etc.
Then move the function.
Test every step.

Read a book on refactoring if you need to.

Code should be easy to read and self explanatory. Comments should not say the same as what the code does, but rather explain why. Function names should say what the code is intended for and the function should only do one thing.
