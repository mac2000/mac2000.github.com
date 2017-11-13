---
layout: post
title: C# WindowsForms load XML from URL into DataGridView

tags: [.net, bindingsource, c#, datagrid, datagridview, dataset, datasource, filter, serialize, table, url, xml, xmlreader, xmlserializer, xmltextreader]
---

I have handler that produces some XML data, that must be loaded into client
aplication datagridview to be filtered.

<amp-img src="/images/wp/dfp_filter.png" alt="screenshot" width="595" height="440"></amp-img>

Here is handler:

```csharp
<%@ WebHandler Language="C#" Class="dfp" %>

using System;
using System.Web;
using System.Collections.Generic;
using RabotaUA.Model;
using Rabota2.DAC;
using System.Xml.Serialization;
using System.Text;
using System.IO;

public class dfp : IHttpHandler {

    public void ProcessRequest (HttpContext context) {
        context.Response.ContentType = "text/xml";

        StringBuilder sb = new StringBuilder();
        StringWriter sw = new StringWriter(sb);
        XmlSerializer serializer = new XmlSerializer(typeof(List<dfp_list_item>));

        serializer.Serialize(sw, get_items());

        context.Response.Write(sb.ToString().Replace("utf-16", "utf-8"));
    }

    public List<dfp_list_item> get_items()
    {
        List<dfp_list_item> items = new List<dfp_list_item>();

        try
        {

            foreach (CityInfo item in CacheHelper.CityList(false))
            {
                items.Add(new dfp_list_item(item.Id, item.Name, "Регион"));
            }

            foreach (RubricInfo item in RubricDAC.GetParentRubricFullList())
            {
                items.Add(new dfp_list_item(item.RubricId, item.RubricName, "Главная рубрика"));
            }

            foreach (RubricInfo item in RubricDAC.GetChildRubricFullList())
            {
                items.Add(new dfp_list_item(item.RubricId, item.RubricName, "Подрубрика"));
            }

            foreach (RabotaUA.Model.SynonymousInfo item in Rabota2.DAC.SynonymousDAC.SearchForAdmin("", 0, 0, 0, int.MaxValue, 0, "", "", ""))
            {
                items.Add(new dfp_list_item(item.Id, item.Name, "PZ"));
            }
        }
        catch (Exception)
        {
        }

        return items;
    }

    public bool IsReusable {
        get {
            return false;
        }
    }

}

public class dfp_list_item
{
    private int _id;

    public int id
    {
        get { return _id; }
        set { _id = value; }
    }
    private string _name;

    public string name
    {
        get { return _name; }
        set { _name = value; }
    }
    private string _type;

    public string type
    {
        get { return _type; }
        set { _type = value; }
    }

    public dfp_list_item() { }
    public dfp_list_item(int id, string name, string type)
    {
        this.id = id;
        this.name = name;
        this.type = type;
    }
}
```

And here is app code to load xml from url into datagridview:

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Text;
using System.Windows.Forms;
using System.Net;
using System.Xml.Serialization;
using System.Xml;
using System.IO;

namespace dfp
{
    public partial class Form1 : Form
    {
        BindingSource bs = new BindingSource();

        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            try
            {
                WebClient wc = new WebClient();
                wc.Encoding = Encoding.UTF8;
                string xml = wc.DownloadString("http://rabota.ua/Export/dfp.ashx");

                XmlSerializer xs = new XmlSerializer(typeof(List<dfp_list_item>));

                StringReader sr = new StringReader(xml);

                XmlTextReader xtr = new XmlTextReader(sr);

                DataSet ds = new DataSet();
                ds.ReadXml(xtr);

                bs.DataSource = ds;
                bs.DataMember = ds.Tables[0].TableName;
                dataGridView2.DataSource = bs;

                dataGridView2.Columns[0].HeaderText = "ID";
                dataGridView2.Columns[1].HeaderText = "Название";
                dataGridView2.Columns[2].HeaderText = "Тип";
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }

        public string build_query()
        {
            List<string> p = new List<string>();

            if (!checkBox1.Checked) p.Add("type <> 'Регион'");
            if (!checkBox2.Checked) p.Add("type <> 'Главная рубрика'");
            if (!checkBox3.Checked) p.Add("type <> 'Подрубрика'");
            if (!checkBox4.Checked) p.Add("type <> 'PZ'");

            if (!string.IsNullOrEmpty(textBox1.Text.Trim())) p.Add("(name LIKE '%" + textBox1.Text.Trim() + "%' OR id LIKE '%" + textBox1.Text.Trim() + "%')");

            return string.Join(" AND ", p.ToArray());
        }

        private void checkBox1_CheckedChanged(object sender, EventArgs e)
        {
            bs.Filter = build_query();

        }

        private void checkBox2_CheckedChanged(object sender, EventArgs e)
        {
            bs.Filter = build_query();
        }

        private void checkBox3_CheckedChanged(object sender, EventArgs e)
        {
            bs.Filter = build_query();
        }

        private void checkBox4_CheckedChanged(object sender, EventArgs e)
        {
            bs.Filter = build_query();
        }

        private void textBox1_TextChanged(object sender, EventArgs e)
        {
            bs.Filter = build_query();
        }
    }

    public class dfp_list_item
    {
        private int _id;

        public int id
        {
            get { return _id; }
            set { _id = value; }
        }
        private string _name;

        public string name
        {
            get { return _name; }
            set { _name = value; }
        }
        private string _type;

        public string type
        {
            get { return _type; }
            set { _type = value; }
        }

        public dfp_list_item() { }
        public dfp_list_item(int id, string name, string type)
        {
            this.id = id;
            this.name = name;
            this.type = type;
        }
    }
}
```
