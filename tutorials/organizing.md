@page organizing Organizing Folders And Files
@parent tutorials 1

The secret to building large apps is to NEVER build
large apps.  Break up your applications into small 
pieces.  Then, assemble those testable, bite-sized pieces 
into your big application.

JavaScriptMVC 3.X is built with this pattern in 
mind. As opposed to a single flat 'scripts' folder, 
JavaScriptMVC breaks up your application into
manageable, isolated modules. This tutorial discusses
the reasons for doing this and how to do it.

## Why

Traditionally, JavaScript, CSS and static resources were seen as second-class
citizens when compared to server code.  JavaScript code was put in a single
flat 'scripts' folder that looked like:

    button.js
    jquery.ui.calendar.js
    contactmanager.js
    tabs.js
    jquery.js
    nav.js
    resizer.js
    \test
      button_test.js
      contactmanager.js
      tabs_test.js
      nav_test.js

This worked ok for a limited amount of JavaScript; however; client code
increasingly represents a larger proportion of an 
application's codebase.  What works for 10 files does not work for 100.

Complicating matters, an individual JavaScript file might have dependencies on
non-JavaScript resources.  It's easy to imagine a menu needing 
a specific stylesheet, images, or [jQuery.View client side template] to run.  

Spreading these dependencies across images, styles, templates etc folders
leads to bad organization and potentially bad performance.  For example, it can be
hard to know if a particular style rule is needed.

### The Fix

JavaScriptMVC gives each resource you author it's own folder.  Typically,
the folder will hold the resource, the resource's demo page, test page,
test script, and any other files specific to that resource.

For example, a tabs folder might look like:

    \tabs
      tabs.js
      tabs.html
      funcunit.html
      tabs_test.js
      tabs.css




## How 

Before we discuss best practices for organizing your application, a little
throat clearing ...

> Every app is different. Providing a single folder structure for
all applications is impossible. However, there are several useful 
patterns that when understood can keep your
application under control. JavaScriptMVC is extremely flexible so use your best judgement.  

This guide walks you through starting with a small-ish example app and where you would add
features over time.


### App and Library Folders

In general, a JavaScriptMVC application is divided into two main folders:

__App Folder__ - houses code specific to a particular 
application.  The code in this folder is very unlikely to be
used in other places.  The folder name reflects the name of the application 
being built.  

Create an application folder and files with

@codestart text
  js jquery\generate\app cms
@codeend

__Library Folders__ - houses general code that 
can be reused across several applications.  It is the perfect place for 
reusable controls like a tabs widget.  Typically folder names reflect
the name of the organization building the controls.  

App folder code typically 'steals' and configures 'library' code.  

### Resource Types

An application is comprised of various resources.  JavaScriptMVC's code generators make
creating these resources easy.

__Model__ - A model represents a set of services.  Typically, they exist within an application
folder's <code>models</code> directory and are used to request data by other controls.

Generate a model like:

@codestart text
  js jquery\generate\model cms\models\image
@codeend

__Controller__ - A controller is a widget or code that combines and organizes
several widgets.  Reusable widgets are added to library folders.  Controllers specific
to an application should be put in a folder within an application folder.

Generate a controller like:

@codestart text
  js jquery\generate\controller jupiter\tabs
@codeend


__Plugin__ - A plugin is a low-level reusable module such as a special event or dom extension.
It does not typically have a visible component.  These should be added to library folders.

@codestart text
  js jquery\generate\plugin jupiter\range
@codeend


## Example Application

The example is a content management system that organizes 'videos', 'images', and 
'articles' under a tabbed layout.  For each content type, the user needs
to be able to edit a selected item of that type.

@image tutorials/cms.png



If the application's name is __cms__ and it is built by Jupiter, a basic version
might look like:

@codestart none
\cms
  \models    - models for the CMS
  \views     - views to configure the grid
  cms.js
\jupiter
  \tabs      - a basic tabs widget
  \edit      - binds a form to edit a model instance
  \grid      - a configurable grid
    \views
@codeend

This basic version assumes that we can configure the grid and edit widget
enough to produces the desired functionality. In this case,
<code>cms/cms.js</code> might look like:

    steal('jupiter/tabs',
          'jupiter/grid',
          'jupiter/create',
          './models/image',
          './models/video',
          './models/article',function(){
      
      $('#tabs').jupiter_tabs();
      
      // Configure the video grid
      $('#videos').jupiter_grid({
        model: Cms.Models.Video,
        view: "//cms/views/videos.ejs"
      })
      
      // listen for when a video is selected
      .bind('select' , function(ev, video){
        
        // update the edit form with the selected 
        // video's attributes
        $('#videoEdit').jupiter_edit({ model: video });
      });
  
      // Do the same for images and articles
      $('#images').jupiter_grid({
        model: Cms.Models.Image,
        view: "//cms/views/images.ejs"
      }).bind('select' , function(ev, image){
        $('#imageEdit').jupiter_edit({ model: image });
      });
      
      $('#articles').jupiter_grid({
        model: Cms.Models.Article,
        view: "//cms/views/article.ejs"
      }).bind('select' , function(ev, article){
        $('#articleEdit').jupiter_edit({ model: article });
      });

    })

Notice that the <code>cms.js</code> configures the grid and edit widgets with
the cms folder's models and views.  This represents an ideal separation between
app specific code and reusable widgets.  However, it's extremely rare that 
widgets are able to provide all the functionality an app needs to meet its
requirements.

### More complexity

Eventually, you won't be able to configure abstract widgets to satisfy
the requirements of your application.  For example, you might need to 
add specific functionality around listing and editing videos (such as a thumbnail editor).

This is application specific functionality and belongs 
in the application folder.  We'll put it in a controller for each type:

@codestart none
\cms
  \articles - functionality in the articles tab
  \images   - functionality in the images tab
  \videos   - functionality in the videos tab
  \models  
  \views     
  cms.js
\jupiter
  \thumbnail
  \tabs     
  \edit      
  \grid      
    \views
@codeend

<code>cms/cms.js</code> now looks like:

    steal('cms/articles',
          'cms/images',
          'cms/videos',
          'jupiter/tabs',
          function(){
      
      $('#tabs').jupiter_tabs();
      
      // add the video grid
      $('#videos').cms_videos()
      
      // Do the same for images and articles
      $('#images').cms_images();
      $('#articles').cms_articles();

    })

<code>cms/articles/articles.js</code> might look like:

    steal('jupiter/grid',
          'jupiter/edit',
          'jquery/controller', 
          'jquery/view/ejs',
          'jupiter/thumbnail',
          function(){
      
      $.Controller('Cms.Articles',
      {
        listensTo: ["select"]
      },
      {
        init : function(){
          this.element.html('//cms/articles/views/init.ejs',{});
          this.find('.grid').jupiter_grid({
	        model: Cms.Models.Article,
	        view: "//cms/articles/views/article.ejs"
	      });
        },
        "select" : function(el, ev, article){
          this.find('.edit').jupiter_edit({ model: article })
          	.find('.thumbs').jupiter_thumbnail();
        }
      });
      
    });

### Adding leaves to the tree

In the previous example, we moved the code in <code>cms/cms.js</code> into
an articles, images, and videos plugin.  Each of these plugins should
work independently from each other, have it's own tests and demo page.

Communication between these high-level
controls should be configured in <code>cms/cms.js</code>.

Essentially, as your needs become more specific, you are encouraged to 
nest plugins within each other.

In this example, after separating out each type into it's own plugin, you might
want to split the type into edit and grid controls.  The resulting 
folder structure might look like:

@codestart none
\cms
  \articles
    \grid
    \edit
  \images
    \grid
    \edit   
  \videos
    \grid
    \edit   
  \models  
  \views     
  cms.js
\jupiter
  \thumbnail
  \tabs     
  \edit      
  \grid      
@codeend

<code>cms/articles/articles.js</code> might now look like:

    steal('cms/articles/grid',
          'cms/articles/edit',
          'jquery/controller',
          'jquery/view/ejs',
          function(){
      
      $.Controller('Cms.Articles',
      {
        listensTo: ["select"]
      },
      {
        init : function(){
          this.element.html('//cms/articles/views/init.ejs',{});
          this.find('.grid').cms_articles_grid();
        },
        "select" : function(el, ev, article){
          this.find('.edit').cms_articles_edit({ model: article });
        }
      });
    })

JavaScriptMVC encourages you to organize your application folder as a tree.
The leaves of the tree are micro-controls that perform a specific task (such as
allowing the editing of videos).  

Higher-order controls (<code>cms/articles/articles.js</code>) combine leaves and other nodes 
into more complex functionality.  The root of the application is the application file 
(<code>cms/cms.js</code>).  It combines and configures all high-level widgets.

In general, low-level controls use jQuery.trigger to send messages 'up' to higher-order
controls.  Higher-order controls typically call methods on lower-level controls

The Cms.Articles control listening to a 'select' event produced by
Cms.Articles.Grid and creating (or updating) the Cms.Articles.Edit control
is a great example of this.

The situation where this breaks down is usually when a 'state' needs to be shared and communicated
across several controls.  [jQuery.Observe] and client [jQuery.Model models] are useful 
for this situation.

## Conclusion

This is an extremely abstract article, but hopefully illustrates a few
important trends of JavaScriptMVC organization:

  - Put code specific to an app in the application folder.
  - Put reusable plugins, widgets, and other code into library folders.
  - Fill out the tree.