# Quick Start Guide #

## Introduction ##

AeliaMDI is a lightweight MDI Framework, based off of AceMDI, that facilitates the creation of professional looking MDI applications.

When developing Swing Applications, `JDesktopPane` and `JInternalFrame` are provided for use with MDI applications. However, `JInternalFrame` has some pretty annoying quirks. For instance, when a `JInternalFrame` is maximized it only occupies the whole area of the desktop pane with its title bar as is. What a general user might expect is that the title bar should vanish and the minimize, maximize and close buttons should appear in the menubar of main application frame.

Due to this, many prefer not use `JInternalFrame` and instead opt for closeable tabs to represent their documents. However, in this approach they lose the facility to place documents side by side in a "restored" state to compare them.

AeliaMDI is designed to solve exactly these two problems. It manages your "views" as closable tabs when maximized and as internal frames when restored or minimized.

AeliaMDI is a fork of the last stable release of AceMDI available. Thanks go out to Pritam G. Barhate for creating such a great MDI library.

Migration from AceMDI to AeliaMDI is pretty straight-forward, but unfortunately there are certain small changes that prevent it from being a drop-in replacement.

## Getting Started ##

So let's get started!

The general steps to getting started are:
  1. Download AeliaMDI
  1. Importing the packages
  1. Dive In!

### Download AeliaMDI ###

You can download the binary distributions at the [AeliaMDI Google Code page](http://code.google.com/p/aeliamdi/downloads/list?q=label:Featured)

If you download the `aeliamdi-with-deps-1.0.0.jar` you do not need to download any dependencies. Otherwise, be sure to download those and include them in your classpath.

Currently AeliaMDI only requires the gdswing library by Guy Davis. If you downloaded the with-deps JAR, you've already got what you need!

### Import Packages ###

Ensure that you have `aeliamdi-1.0.0.jar` in your `CLASSPATH`.

### Dive In! ###
#### Overview ####

When dealing with AeliaMDI you will mainly use two classes.

| `MDIFrame` | This class extends JFrame to provide a main frame for the application which is "MDI aware." You use `MDIFrame.add(MDIView view)` method to add a new view to it. It provides `JDesktopPane`-like faclities to your application. |
|:-----------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `MDIView`  | This class provides the you an "MDI aware" container. It provides you similar facilities to `JInternalFrame`.                                                                                                                   |

#### `MDIFrame` ####
It is same as creating any other JFrame.

MDIFrame myFrame = new MDIFrame("title");

#### `MDIView` ####
Now let's create an MDIView and add it to our MDIFrame!

You might add this code to your File > New menu handler's `actionPerformed` method:
```
MDIView aView = new MDIView( parentMDIFrame, defaultComponent, title, icon );
```

Here if title is null, the framework provides automatic titles such as Untitled1, Untitled2, etc. `defaultComponent` is any object of any class that extends Component class. AeliaMDI will try to hand-off focus to this component whenever the `MDIView` gets focus. For example,
```
//create a view of your document
JTextPane textPane= new JTextPane();
JScrollPane scrollPane = new JScrollPane(textPane);
MDIIcon icon = new MDIIcon("new.gif");
//create a MDIView
MDIView text = new MDIView(editorFrame, textPane, null, icon);
//add a MDIViewListener to listen MDIViewEvents.
text.addMDIViewListener(new aMDIViewListener());
//set a layout for the view and add the component to it
text.setLayout(new BorderLayout());
text.add(scrollPane, BorderLayout.CENTER);
//add the view to your MDIFrame
myFrame.add(text);
```

In the above code whenever the user switches to the `MDIView`, `textPane` receives focus.

#### `MDIViewListener` ####
You will probably want to be listening to events that happen to your various `MDIView`s, so let's setup some `MDIViewListener`s.

`MDIViewListener` listens for `MDIViewEvent`s which is similar to `InternalFrameEvent`. By registering a MDIViewListener you can react to the MDIView events.

In code shown above, the line
```
text.addMDIViewListener(new aMDIViewListener());
```
registers a listener to the MDIView.

The provided events are
```
MDIVIEW_ACTIVIATED
MDIVIEW_DEACTIVIATED
MDIVIEW_OPENED
MDIVIEW_CLOSING
MDIVIEW_CLOSED
MDIVIEW_ICONIFIED
MDIVIEW_RESTORED
MDIVIEW_MAXIMIZED
```

Using respective methods provided in `MDIViewListener` class you can provide the code that reacts to that specific event.

When a `MDIView` is maximized, only that view will receive	a maximized event. But all the views present in the same `MDIFrame` are transformed to maximized state due to the inherent nature of the framework which says that when one view is maximised all the views should be represented as tabs. Exactly reverse to this, when one of the views is restored all other views are restored or minimized (in exact words, to the state that they were in before going into maximized state). But only the view for which the restored button was clicked will receive the restored event.

Also note that when the main `MDIFrame` for the views is closed, no view will receive the `MDIView` closing or closed events. Instead you will have to deal with them manually in a `windowClosing` event listener on the `MDIFrame`. You can obtain the `MDIView`s contained in an `MDIFrame` by calling the `getViews()` method. You may then call `closeView()` on all the views sequentially.

#### Usage with `JSplitPane` ####

In some cases you might want to use a JSplitPane in your `MDIFrame`. This requires a slight adjustment to the behavior of `MDIFrame`. `MDIFrame` manages all the views in a `JPanel` variable known as `viewContainer`.
```
JPanel viewContainer = editorFrame.getViewContainer();
editorFrame.getContentPane().remove(viewContainer);
JSplitPane splitPane = new JSplitPane(JSplitPane.VERTICAL_SPLIT, viewContainer, anotherComponent);
```

In the above code, first we obtain the `viewContainer` by using `MDIFrame.getViewContainer()` then we remove it from the `MDIFrame` and add it to the `JSplitPane`.

#### `MDIFrameListener` ####
If you are interested in knowing when the `viewPane` changes from `TABS` to `DESKTOP` you'll be using a MDIFrameListener.

`MDIFrame.addMDIFrameListener(MDIFrameListener listener)`

### Further Documentation ###
For detailed information on each class please see the JavaDocs of the AeliaMDI framework. Also looking at the source code of `MDITester.java in` `examples` folder will increase your understanding of working and possible uses of various methods provided by the framework.

### Supporting a Custom Look and Feel ###

AeliaMDI tries to achive the best results with a look and feel unknown to it, but a little bit of touching up might be required. You can do this by extending the MDIFrameButton class and then call:
```
MDIFrame.setCloseButton()
MDIFrame.setRestoreButton()
MDIFrame.setIconifyButton() 
```
You will have to pass them the instances of your custom class. Be sure to take look at
`CustomMDIButton.java`
in the `examples` folder before reading any further.

Basically all you have to do is call `super(type, frame)` in your constructor. AeliaMDI obtains the icons for the buttons using
```
UIManager.getIcon("InternalFrame.iconifyIcon"),	
UIManager.getIcon("InternalFrame.iconifyIcon"), 
UIManager.getIcon("InternalFrame.closeIcon")
```
methods. You can obtain them the same way if look and feel designer has taken care to support these properties correctly. But otherwise, you will have to create icons which resemble the icons provided by the look and feel and load them in instance variables in the constructor.
Then you have to override the `setupLookAndFeel()` feel method, following way.

```
public void setupLookAndFeel( String ID )
{
	if( ID.equals( "ID of your look and feel" ) )
	{
		//Set your look and feel specific properties
	}
	else
	{       //In all other cases use superclass functionality.	
		super.setupLookAndFeel( ID );
	}
}
```

If you don't know the id of your look and feel, then you can call the below when your look and feel is set.
System.out.println(UIManager.getLookAndFeel().getID());```