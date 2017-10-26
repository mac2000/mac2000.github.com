---
layout: post
title: Asp.Net move all Javascript to page closure and css to head

tags: [.net, asp.net, c#, ControlDesigner, css, DefaultProperty, GetDesignTimeHtml, HtmlGenericControl, HtmlTextWriter, InitComplete, javascript, OnPreRender, RenderControl, StringWriter, ToolboxData]
---

Here is my use case:

In our project we have hungreds of controls, and each of them has some styles and javascripts.

I's very convenient coz I can change things very fast, but it's very bad for page load speed.

So we need ability to move all our code in special places.

What I've done is create Custom Control that inherites from PlaceHolder, but rather than rendering itself inplace it renders into page closure (for js) or head (for css).

Here is how its look like:

```csharp
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Default.aspx.cs" Inherits="AssetsPlaceHolder._Default" %>

<%@ Register Assembly="Rabota2.Controls" Namespace="Rabota2.Controls" TagPrefix="rua" %>
<%@ Register src="Control1.ascx" tagname="Control1" tagprefix="uc1" %>
<!DOCTYPE HTML>
<html lang="ru">
<head runat="server">
    <title>AssetsPlaceHolder</title>
</head>
<body>
    <form id="form1" runat="server">
        <h3>Assets Place Holder</h3>
        <rua:CssPlaceHolder ID="CssPlaceHolder1" runat="server">
            <style type="text/css">
                body, .<%= Control11.ClientID%> {
                    color:#f00;
                }
            </style>
        </rua:CssPlaceHolder>
        <uc1:Control1 ID="Control11" runat="server" />
    </form>
</body>
</html>
```

And here is example of control code:

```csharp
<%@ Control Language="C#" AutoEventWireup="true" CodeBehind="Control1.ascx.cs" Inherits="AssetsPlaceHolder.Control1" %>
<%@ Register Assembly="Rabota2.Controls" Namespace="Rabota2.Controls" TagPrefix="rua" %>

<p>Hello From Control 1</p>
<rua:JavaScriptPlaceHolder ID="JavaScriptPlaceHolder1" runat="server">
<script>
    alert('hello <%= this.ClientID%>');
</script>
</rua:JavaScriptPlaceHolder>
```

Without our custom placeholder styles and scripts will be rendrered in their places, but with our code they will be rendered in page closere and header, here is rendered html:

```html
<!DOCTYPE HTML>

<html lang="ru">

<head><title>

    AssetsPlaceHolder

</title><style id="rua-css-js">

                body, .Control11 {

                    color:#f00;

                }

        </style><script id="rua-head-js"></script></head>

<body>

    <form name="form1" method="post" action="" id="form1">

<div>

<input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="/wEPDwULLTIwMDgxNjI2MDEPZBYEAgEPZBYCAgEPFgIeCWlubmVyaHRtbAV+DQogICAgICAgICAgICANCiAgICAgICAgICAgICAgICBib2R5LCAuQ29udHJvbDExIHsNCiAgICAgICAgICAgICAgICAgICAgY29sb3I6I2YwMDsNCiAgICAgICAgICAgICAgICB9DQogICAgICAgICAgICANCiAgICAgICAgZAIDD2QWAgIFDxYCHwAFJQ0KDQogICAgYWxlcnQoJ2hlbGxvIENvbnRyb2wxMScpOw0KDQpkZOBnZ28jerbTEhM0uBSmjX6KuQCa6Co5bwXXdVV5yALk" />

</div>

        <h3>Assets Place Holder</h3>

<p>Hello From Control 1</p>

    <script id="rua-closure-js">

    alert('hello Control11');

</script></form>

</body>

</html>
```

And here is how this will look like in VisualStudio:

![screenshot](/images/wp/assetsplaceholder.png)

And here is code doing all this:

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;
using System.Web.UI;
using System.Web.UI.Design;
using System.Web.UI.HtmlControls;
using System.Web.UI.WebControls;

namespace Rabota2.Controls
{
    [ToolboxData("<{0}:JavaScriptPlaceHolder runat=\"server\"><script type=\"text/javascript\"></script></{0}:JavaScriptPlaceHolder>")]
    [DefaultProperty("Holder")]
    public class JavaScriptPlaceHolder : AssetsPlaceHolder
    {
        public enum AssetHolder
        {
            Header,
            Closure
        }
        protected AssetHolder _holder = AssetHolder.Closure;
        [Category("Appearance")]
        public AssetHolder Holder
        {
            get { return _holder; }
            set { _holder = value; }
        }

        protected override HtmlGenericControl GetHolder()
        {
            return Holder == AssetHolder.Header ? JS_HEAD : JS_CLOSURE;
        }
    }

    [ToolboxData("<{0}:CssPlaceHolder runat=\"server\"><style type=\"text/css\"></style></{0}:CssPlaceHolder>")]
    public class CssPlaceHolder : AssetsPlaceHolder
    {
        protected override HtmlGenericControl GetHolder()
        {
            return CSS_HEAD;
        }
    }

    #region Abstract AssetsPlaceHolder
    [Designer(typeof(AssetsPlaceHolderDesigner))]
    public abstract class AssetsPlaceHolder : PlaceHolder
    {
        /// <summary>
        /// Element IDs, will be used to create elements only once, look into page_InitComplete
        /// </summary>
        protected readonly string JS_HEAD_ID = "rua-head-js";
        protected readonly string JS_CLOSURE_ID = "rua-closure-js";
        protected readonly string CSS_HEAD_ID = "rua-css-js";

        /// <summary>
        /// Elements that will contain all css and javascript from page
        /// </summary>
        protected HtmlGenericControl JS_HEAD;
        protected HtmlGenericControl JS_CLOSURE;
        protected HtmlGenericControl CSS_HEAD;

        /// <summary>
        /// Abstract method to get one of elements defined above, look into implementation in JavaScriptPlaceHolder and CssPlaceHolder
        /// </summary>
        /// <returns>HtmlGenericControl</returns>
        protected abstract HtmlGenericControl GetHolder();

        /// <summary>
        /// On constructor we're going to attach to page init complete event out custom handler that will create holders for code
        /// </summary>
        public AssetsPlaceHolder()
            : base()
        {
            //Run this code only if we have context (this will prevent visual studio desing mode errors)
            if (Context != null)
            {
                Page page = (Page)System.Web.HttpContext.Current.Handler;
                page.InitComplete += new EventHandler(page_InitComplete);
            }
        }

        private void page_InitComplete(object sender, EventArgs e)
        {
            //Run this code only if we have context (this will prevent visual studio desing mode errors)
            if (Context != null)
            {
                //Add css holder to head, if it already not here
                CSS_HEAD = (HtmlGenericControl)Page.Header.FindControl(CSS_HEAD_ID);
                if (CSS_HEAD == null)
                {
                    CSS_HEAD = new HtmlGenericControl("style");
                    CSS_HEAD.ID = CSS_HEAD_ID;
                    Page.Header.Controls.Add(CSS_HEAD);
                }

                //Add js holder to head, if it already not here
                JS_HEAD = (HtmlGenericControl)Page.Header.FindControl(JS_HEAD_ID);
                if (JS_HEAD == null)
                {
                    JS_HEAD = new HtmlGenericControl("script");
                    JS_HEAD.ID = JS_HEAD_ID;
                    Page.Header.Controls.Add(JS_HEAD);
                }

                //Add js holder to page closure, if it already not here
                JS_CLOSURE = (HtmlGenericControl)Page.FindControl(JS_CLOSURE_ID);
                if (JS_CLOSURE == null)
                {
                    JS_CLOSURE = new HtmlGenericControl("script");
                    JS_CLOSURE.ID = JS_CLOSURE_ID;
                    Page.Form.Controls.Add(JS_CLOSURE);
                }
            }
        }

        /// <summary>
        /// Here is magic starts - override RenderControl to do nothing, so in place where out control lives will be print nothing
        /// </summary>
        /// <param name="writer"></param>
        public override void RenderControl(HtmlTextWriter writer) { }

        /// <summary>
        /// Magic end - override OnPreRender event to move rendered control content to selected holder
        /// </summary>
        /// <param name="e"></param>
        protected override void OnPreRender(EventArgs e)
        {
            HtmlGenericControl holder = GetHolder(); // GetHolder() must be implemented in child classes

            //Run this code only if we have context (this will prevent visual studio desing mode errors)
            if (Context != null)
            {
                if (holder != null)
                {
                    //We're using base.RenderControl() to render control into string rather than page
                    string innerHtml = string.Empty;
                    using (StringWriter stringWriter = new StringWriter())
                    {
                        using (HtmlTextWriter htmlTextWriter = new HtmlTextWriter(stringWriter))
                        {
                            base.RenderControl(htmlTextWriter);
                            innerHtml = stringWriter.ToString();
                        }
                    }

                    //For normal VisualStudio syntax highlighting we're using <scrtip>...</script> and <style>...</style> tags inside our placeholders, and must strip them here
                    innerHtml = Regex.Replace(innerHtml, @"<\/?(script|style)[^>]*>", "", RegexOptions.IgnoreCase | RegexOptions.Singleline);

                    holder.InnerHtml += innerHtml;
                }
            }
            base.OnPreRender(e);
        }
    }
    #endregion

    #region Designer
    /// <summary>
    /// Assets PlaceHolder ControlDesigner
    ///
    /// This designer simply renders label indicating which assets placeholder used
    /// </summary>
    public class AssetsPlaceHolderDesigner : ControlDesigner
    {
        public override string GetDesignTimeHtml()
        {
            Label label = new Label();
            label.Text = "asset";

            if (Component.GetType() == typeof(JavaScriptPlaceHolder)) label.Text = "js";
            else if (Component.GetType() == typeof(CssPlaceHolder)) label.Text = "css";

            label.BackColor = ColorTranslator.FromHtml("#444444");
            if (Component.GetType() == typeof(JavaScriptPlaceHolder)) label.BackColor = ColorTranslator.FromHtml("#CC0000");
            else if (Component.GetType() == typeof(CssPlaceHolder)) label.BackColor = ColorTranslator.FromHtml("#006E2E");

            label.ForeColor = ColorTranslator.FromHtml("#EEEEEE");
            if (Component.GetType() == typeof(JavaScriptPlaceHolder)) label.BackColor = ColorTranslator.FromHtml("#CC0000");
            else if (Component.GetType() == typeof(CssPlaceHolder)) label.BackColor = ColorTranslator.FromHtml("#006E2E");

            label.Style.Add("padding", "0 0.5em");

            string html = string.Empty;
            using (StringWriter stringWriter = new StringWriter())
            {
                using (HtmlTextWriter htmlTextWriter = new HtmlTextWriter(stringWriter))
                {
                    label.RenderControl(htmlTextWriter);
                    html = stringWriter.ToString();
                }
            }

            return html;
        }
    }
    #endregion
}
```

TODO:

* Add AppSettings configuration option to turn on\off this feature
* Add public boolean option Minify
* Add ability to attach files by path
