---
layout: post
title: WinForms WebBrowser DocumentCompleted

tags: [.net, c#, webbrowser, winforms, documentcompleted, oermissionset, allowwebbrowserdrop, iswebbrowsercontextmenuenabled, objectforscripting, scripterrorssuspend]
---

WebBrowser can give you fake `DocumentCompleted` event, it do not count iframes, counters etc. Here is how to fix it:

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Text;
using System.Windows.Forms;
using System.Security.Permissions;
using mshtml;
using System.IO;

namespace HeadHunterNewCompanies
{
    [PermissionSet(SecurityAction.Demand, Name = "FullTrust")]
    [System.Runtime.InteropServices.ComVisibleAttribute(true)]
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();

            webBrowser1.AllowWebBrowserDrop = false;
            webBrowser1.IsWebBrowserContextMenuEnabled = false;
            webBrowser1.WebBrowserShortcutsEnabled = false;
            webBrowser1.ObjectForScripting = this;
            // Uncomment the following line when you are finished debugging.
            webBrowser1.ScriptErrorsSuppressed = true;
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            webBrowser1.Navigate("http://hh.ua/employersList.do");
        }

        private void webBrowser1_DocumentCompleted(object sender, WebBrowserDocumentCompletedEventArgs e)
        {
            if (e.Url.AbsolutePath != (sender as WebBrowser).Url.AbsolutePath)
                return;

            HtmlElement head = webBrowser1.Document.GetElementsByTagName("head")[0];
            HtmlElement scriptEl = webBrowser1.Document.CreateElement("script");
            IHTMLScriptElement element = (IHTMLScriptElement)scriptEl.DomElement;
            element.text = File.ReadAllText("jquery-1.4.2.min.js");
            head.AppendChild(scriptEl);

            scriptEl = webBrowser1.Document.CreateElement("script");
            element = (IHTMLScriptElement)scriptEl.DomElement;
            element.text = "function myfunc() { var lastPageNumber = $('.b-pager-lite a:last').text(); alert(lastPageNumber); }";
            head.AppendChild(scriptEl);
            webBrowser1.Document.InvokeScript("myfunc");
            //
        }
    }
}
```
