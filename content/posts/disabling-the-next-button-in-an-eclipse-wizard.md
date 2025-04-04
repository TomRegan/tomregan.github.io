+++
date = '2012-12-09'
title = 'Disabling the Next Button in an Eclipse Wizard'
+++

I've been writing an Eclipse wizard that will be used to create new 
socket applications. The options selected in early in the wizard
determine whether it's appropriate for the user to configure connection
properties before creating the project.

If you have written a wizard, chances are you're familiar with the
`setPageComplete(complete:boolean)` method that is used to enable the 
'Finish' button when enough user input has been gathered.

It's more complicated to disable the 'Next' button. To do that, you
have to override the `WizardPage` base implementation of
`getNextPage()` (of course you do, it's 
_totally_ different from the 'Finish' button, isn't it?); this method 
should return `null` to indicate that the next page is not available.

The implementation looks a little bit like this:

{% highlight java %}
class ConfigurationPage extends WizardPage

    ...

    @Override
    public IWizardPage getNextPage()
    {
        if (!isConnectionPageRequired()) {
            return null;
        }
        return super.getNextPage();
    }
}
{% endhighlight %}

Now, depending on the configuration options, we can choose whether to 
enable 'Next' or 'Finish':

![](/assets/images/2012-08-04-disabling-the-next-button-in-an-eclipse-wizard/next.png)

"_He chose, [poorly](http://www.youtube.com/watch?v=Ubw5N8iVDHI)_."

![](/assets/images/2012-08-04-disabling-the-next-button-in-an-eclipse-wizard/finish.png)

He didn't really choose poorly, I just love that scene. And if you're 
wondering, yes, there is a method called `getPreviousPage()`,
although if you plan to disable the 'Back' button you might be a 
sociopath.

The relevant Eclipse doc is [here](http://help.eclipse.org/indigo/index.jsp?topic=%2Forg.eclipse.platform.doc.isv%2Freference%2Fapi%2Forg%2Feclipse%2Fjface%2Fwizard%2FIWizardPage.html).
