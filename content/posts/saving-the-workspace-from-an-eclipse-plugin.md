+++
date = '2012-09-17'
title = 'Saving the Workspace from an Eclipse Plugin'
+++

Since I'm going to the trouble of making a plugin for Eclipse,
I thought I may as well try and make if look and feel a bit
Eclipsy, so when it came to writing an export feature to bundle the
user's project, I decided it would be a jolly good thing if the user
got a dialog to prompt them to save their changes, much as when you run
an application in Eclipse.

The following method will check each window for open editors, check if
the editor has unsaved content, and if one does present a dialog.

{% highlight java %}
private boolean checkForUnsavedFiles() {
    IWorkbenchWindow[] windows = PlatformUI.getWorkbench().getWorkbenchWindows();
    for (IWorkbenchWindow window : windows) {
        IWorkbenchPage page = window.getActivePage();
        IEditorReference[] references = page.getEditorReferences();
        for(IEditorReference reference : references) {
            if (reference.isDirty()) {
                int dialogResponse = unsavedFilesDialog.open();
                if (dialogResponse == YES) {
                    page.saveAllEditors(false);
                    return true;
                } else if (dialogResponse == NO) {
                    return true;
                }
                return false;
            }
        }
    }
    return true;
}
{% endhighlight %}

I also think it's a jolly good thing that users are bullied and cajoled
until their code compiles correctly and efficiently (because I'm a
hypocrite), so to that end I've also written some code to check the project
for annotations (Eclipse refers to errors, bookmarks -- any mark you see
in the left editor gutter -- as an annotation) and to determine if any of
those annotations represent errors (as opposed to warnings, or something else
all-together).

This method searches a list of annotations from the project, checks them
for severity, and if any errors are found shows a dialog.

{% highlight java %}
private boolean checkForCompileErrors(IJavaProject selectedProject) {
    IProject project = selectedProject.getProject();
    IMarker[] markers = null;
    try {
        markers = project.findMarkers(IMarker.PROBLEM, true, IResource.DEPTH_INFINITE);
        for (IMarker marker : markers) {
            Integer severity = (Integer) marker.getAttribute(IMarker.SEVERITY);
            if (severity != null && severity >= IMarker.SEVERITY_ERROR) {
                int dialogResponse = compilationErrorsDialog.open();
                if (dialogResponse == YES) {
                    return true;
                }
                return false;
            }
        }
    } catch (CoreException e) {
        return true;
    }
    return true;
}
{% endhighlight %}

I implemented my own dialogs, but it's worth being aware of the pre-baked
JFace dialogs provided; there's a list [here](https://sureshkrishna.wordpress.com/2008/03/15/jface-dialogs-which-one-is-right-for-you/).

Here are a couple of links to the more interesting pages of Eclipse
documentation:

* [Dialogs in Eclipse](http://help.eclipse.org/indigo/index.jsp?topic=%2Forg.eclipse.platform.doc.isv%2Freference%2Fapi%2Forg%2Feclipse%2Fjface%2Fdialogs%2FDialog.html)
* [IMarker interface](http://help.eclipse.org/indigo/index.jsp?topic=%2Forg.eclipse.platform.doc.isv%2Freference%2Fapi%2Forg%2Feclipse%2Fcore%2Fresources%2FIMarker.html) (used to check for errors)
