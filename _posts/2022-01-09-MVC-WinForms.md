Old apps are often useful but difficult to replace. Often they become this huge mess of tangled code that no one can fix without fear of introducing more bugs. You will hear programmers say, "This app needs to be re-written".

**MVC is the simplest way** to split the code out of a WinForms app with minimal impact. You can do one small part at a time. There are other ways i.e. you can use objects to keep code separate, but often they can become part of the problem.

**WHY?** The reason to refactor becomes apparent when you look at an old WinForms app. Ok some are beautiful, but its rare.

This small project [interval refactor project](https://github.com/richard-porteous/interval-refactor-project ".net C# WinForms MVC style") takes a WinForms C# app and adds MVC design to it. The app has a very simple UI. It doesn't use a database, so you wont see much in the model.

The objective here is to separate UI code from logic and from database queries.

1. WinForms is the UI
2. controller class
3. model class.

*UI Class. (WinForms)* View or display. To keep this simple we let the forms load as normal. On an existing app this can be changed once your code is cleaner. The code behind deals with UI related things, and it makes calls to the controller when something on the UI is changed by the user.

*Controller Class.* Called by UI and Calls the UI. Runs logic, calculations and to update the views with data. Often in these changes you will see that these apps are UI and Database with very little between. Its our job to separate the fact from the fiction. As you will see in later blogs.

*Model Class.* Database code not much else.

This project simply shows you how to add the controller and model, nothing more. See the project readme for more explanation. This can be added without changing any other code and without affecting any behavior. The refactoring of the code is dealt with later.

If you have an existing project all you will need to do to start is create the controller and model for each WinForm, except when a WinForm is there to support - i.e. "Customers" and "CustomerAdd" are two different WinForms but both *could possibly* use the same controller and model. Here are a few snippets from the project.

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

```

In the Controller - we add and initialize the model, and use the passed view. We could use object for the view if sharing or use a separate constructor. Be careful, only share if it will make things better.

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

View the repository to see the example code in action - I used visual studio community, C#, and .NET 5. Modify as you please.

The example app shows one way how you could use MVC. The purist may take it further by starting the controller not the view. You may not want to use the update view function. Its up to you. I would investigate he MVC model further, it could transform your style.  

WinForms for .NET 5 & 6 no longer supports the SQL reporting. BI is available, or you could try [FASTREPORT](https://fastreports.github.io/FastReport.Documentation/ "Fast-Report") which has a community version and is available via NuGet. Whatever you do, make a separate project for your reports under the same solution, to make future decisions about switching easier. If you have an older project I'm planning a Blog to show you how to separate the reports into a separate project. This is likely the best place to start when wanting to renew those .NET 4.6 and earlier projects.

That's it for today. If you see any bugs in the repository feel free to drop me a note. The code is free to use, the repository can be forked or downloaded.


###General

Code should be easy to read and self explanatory. Comments should not say the same as what the code does, but rather explain why. Function names should say what the code is intended for and the function should only do one thing.
