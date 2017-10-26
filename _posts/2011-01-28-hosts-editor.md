---
layout: post
title: Hosts editor

tags: [.net, access, admin, administrator, c#, host, hosts, privilegies, system32, uac]
---

![screehshot](/images/wp/hosts.png)

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Windows.Forms;
using System.IO;
using System.Text.RegularExpressions;

namespace Hosts
{
    public partial class Form1 : Form
    {
        private string _hostsFilePath = string.Format("{0}\\drivers\\etc\\hosts", System.Environment.SystemDirectory);

        public Form1()
        {
            if (!File.Exists(_hostsFilePath))
            {
                MessageBox.Show("Hosts file not found at: " + _hostsFilePath, "ERROR", MessageBoxButtons.OK, MessageBoxIcon.Error);
                Application.Exit();
            }

            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            try
            {
                string hostFileText = File.ReadAllText(_hostsFilePath);
                MatchCollection hostMatches = Regex.Matches(hostFileText, @"^[\t\s ]*(\d+\.\d+\.\d+\.\d+)[\t\s ]+(.*)$", RegexOptions.Multiline);
                foreach (Match hostMatch in hostMatches)
                {
                    string ip = hostMatch.Groups[1].Value.Trim();
                    string domain = hostMatch.Groups[2].Value.Trim();
                    hostsDataSet.Hosts.AddHostsRow(ip, domain);
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message, "ERROR", MessageBoxButtons.OK, MessageBoxIcon.Error);
                Application.Exit();
            }
        }

        private void Form1_FormClosing(object sender, FormClosingEventArgs e)
        {
                try
            {
            if (hostsDataSet.HasChanges())
            {
                if (DialogResult.Yes == MessageBox.Show("You've made some changes, do you want save them?", "Save changes?", MessageBoxButtons.YesNo, MessageBoxIcon.Question))
                {
                    List<string> hosts = new List<string>();
                    foreach (DataRow row in hostsDataSet.Hosts.Rows)
                    {
                        hosts.Add(string.Format("{0,-16}{1}", row[0], row[1]));
                    }

                    string newHostsFileText = string.Join(Environment.NewLine, hosts.ToArray());

                    File.WriteAllText(_hostsFilePath, newHostsFileText);
                }
            }
            }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message, "ERROR", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    Application.Exit();
                }
        }
    }
}
```

For UAC. You must add application manifest file, and change there

```xml
<requestedExecutionLevel level="requireAdministrator" uiAccess="false" />
```

[Hosts](/images/wp/Hosts.zip)
