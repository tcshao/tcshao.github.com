---
layout: post
title: "Teaching an Old Dog New Tricks (MVP and Windows Forms)"
description: ""
category: development
tags: [software design, C#]
---
Windows Forms, Hell yeah!

I've spent the majority of the last decade or so working in Windows Forms.  Winforms may be old, but it's still a very productive platform. 

Patterns and practicies must change as technology does.  With the C#/XAML based apps we have the [Model-View-ViewModel](http://en.wikipedia.org/wiki/Model_View_ViewModel) pattern.  There is also the [Model-View-Controller](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) pattern used in both client-side([Ember](http://emberjs.com/), [AngularJS](http://angularjs.org/)) and server-side([MVC](http://www.asp.net/mvc), [Ruby on Rails](http://rubyonrails.org/)) web frameworks.  The goals of these frameworks is to seperate the responsibilities of retrieving data, processing data, and the actual displaying of the data. 

The "Model" referred to in these patterns is the representation of the data you are trying to display.  The "View" is the thing the end user interacts with.  The third element in these listed patterns refer to how the Model and View work together.  In the MVP pattern, there is a presenter that acts as a middle-man between the View and the Model.  All calls on the UI are routed through the presenter, so there is UI logic inside of the presenter.  In the MVC Pattern, your call is to the controller initially, and the controller decides which View to load and returns it to you.  In the MVVM Pattern, The third confusingly-named object is the 'ViewModel' uses binding and notifications to coordinate the Model and the View.  (See Diagram on [MSDN](http://msdn.microsoft.com/en-us/library/hh848246.aspx))

Model-Vew-Controller
----
The one that we can apply rather easily to a Windows Forms application is the Model-View-Controller Pattern (Or, as a more granular designation, [Passive View](http://martinfowler.com/eaaDev/PassiveScreen.html)).  The reason for this is the plumbing just isn't there in windows forms to do MVVM cleanly. Also, the MVC pattern is more applicable to applications that don't have persisted state.  Also, being that ASP .NET Web Forms is similiar in architecture to Windows Forms, this pattern works well for that too.

I will show a skeleton of an app to show the basics of how an MVC Windows Form app would be structured.  let's say theres a form that lists teachers, and allows you to view students when a particular teacher is selected from a dropdown.

Here's our Model:

    public class Course
    {
        public string Teacher { get; set; }
        public List<string> Students { get; set; }
    }

    public class Model
    {
        public List<Course> Courses { get; set; }

        public Model()
        {
            Courses = new List<Course>
              {
                  new Course {Teacher = "Mr. Smith",
                  	Students = new List<string> {"Student A", "Student B"}},
                  new Course {Teacher = "Mr. Jones", 
                  	Students = new List<string> {"Student A", "Student C"}},
                  new Course {Teacher = "Mr. Bill", 
                  	Students = new List<string> {"Student D", "Student E"}}
              };
        }
    }

OK, now we think about what we display to the user.  We know when the form loads we will have a list box populated with teacher names, and when a teacher is selected, the students will be displayed.  So we know the only requests to our model will be on initialization, and when a teacher changes.  This maps to the following code on the presenter:

    class CoursePresenter
    {
        private readonly ICourseView _view;
        private readonly Model _model;

        public CoursePresenter(ICourseView view)
        {
            _view = view;
            _model = new Model();

            _view.ShowTeachers(_model.Courses
                    .Select(c => c.Teacher)
                    .ToList());
        }

        public void TeacherChanged(string teacherName)
        {
            _view.ShowStudents(_model.Courses
                .Where(c => c.Teacher == teacherName)
                .SelectMany(c => c.Students)
                .ToList());
        }
    }

The ICourseView is the actual Windows Form will we pass into this presenter.  The interface is these two methods:

    public interface ICourseView
    {
        void ShowTeachers(List<string> teachers);
        void ShowStudents(List<string> students);
    }

In the MVP pattern, the Presenter is aware of the view, so we can directly call methods on the view.  If there was any business rules, or database logic, it would be in the Presenter, this leaves us with a view which is just a shell that delegates it's calls:

    public partial class CourseView : Form, ICourseView
    {
        public CourseView()
        {
            InitializeComponent();
            _presenter = new CoursePresenter(this);
        }

        private void TeacherComboBoxSelectedIndexChanged(object sender, EventArgs e)
        {
            _presenter.TeacherChanged(TeacherComboBox.SelectedItem.ToString());
        }

        public void ShowTeachers(List<string> teachers)
        {
            teachers.ForEach(c => TeacherComboBox.Items.Add(c));
        }

        public void ShowStudents(List<string> students)
        {
           StudentsListBox.Items.Clear();
           students.ForEach(s => StudentsListBox.Items.Add(s));
        }
    }

Final Thoughts
----
This is meant to be a guideline of where your boundries are and what objects have a reference to which other ones.  It seems everytime I refactor an app to this I have to refresh my memory again.  As an added bonus, you will now be able to write some unit tests for your presenter.  