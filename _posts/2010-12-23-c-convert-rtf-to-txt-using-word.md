---
layout: post
title: C# convert rtf to txt using word

tags: [.net, c#, convert, doc, docx, lib, microsoft, office, rtf, tool, word]
---

Необходимо было сконвертировать порядка 20 тыс. файлов.

Либы конвертеры - дают плохой результат.

Единственным нормальным решением оказалось: програмно открывать в Word'е документ и сохранять его как текстовый файл.

Далее вырезки кода:

```csharp
public List<string> file_pathes = new List<string>();
public int total_files = 0;

public void r(string path)
{
    string[] pathes = Directory.GetFiles(path);
    foreach (string p in pathes)
    {
        file_pathes.Add(p);
    }

    pathes = Directory.GetDirectories(path);
    foreach (string p in pathes)
    {
        r(p);
    }
}

private void button1_Click(object sender, EventArgs e)
{
    r(@"C:\Users\mac\Desktop\2convert");

    total_files = file_pathes.Count;

    //OBJECT OF MISSING "NULL VALUE"
    Object oMissing = System.Reflection.Missing.Value;

    //OBJECTS OF FALSE AND TRUE
    Object oTrue = true;
    Object oFalse = false;

    //CREATING OBJECTS OF WORD AND DOCUMENT
    Microsoft.Office.Interop.Word.Application oWord = new Microsoft.Office.Interop.Word.Application();
    Microsoft.Office.Interop.Word.Document oWordDoc = new Microsoft.Office.Interop.Word.Document();

    //SETTING THE VISIBILITY TO TRUE
    oWord.Visible = false;

    int c = 1;
    DateTime dts = DateTime.Now;
    foreach (string f in file_pathes)
    {
        string dir = Path.GetDirectoryName(f);
        string n = Path.GetFileNameWithoutExtension(f);
        string ext = Path.GetExtension(f);
        string np = Path.Combine(dir, n + ".txt");

        if (n == "Business Growth Hypothesis") continue;

        if (ext != "txt")
        {
            long size = 0;
            if (File.Exists(np))
            {
                FileInfo fInfo = new FileInfo(np);
                size = fInfo.Length;
            }

            if (!File.Exists(np) || size == 0)
            {
                try
                {
                    Microsoft.Office.Interop.Word.Document oDoc = oWord.Documents.Open(f, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing);
                    oDoc.SaveAs2(np, Microsoft.Office.Interop.Word.WdSaveFormat.wdFormatText, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing);
                    oDoc.Close(oMissing, oMissing, oMissing);
                }
                catch (Exception ex)
                {
                    textBox2.Text += f + Environment.NewLine;
                    //MessageBox.Show(f, "ERROR");
                }
            }
        }

        c++;
        DateTime dtc = DateTime.Now;
        int p = c / (total_files / 100);

        TimeSpan dtts = dtc.Subtract(dts);
        double s = dtts.Seconds;
        if (s == 0) s = 1;
        double sr = (total_files - c) * (c / s);

        TimeSpan ddd = DateTime.Now.AddSeconds(sr).Subtract(DateTime.Now);

        string m = string.Format("{1}/{0} ({2}%)", total_files, c, p);
        m += Environment.NewLine;
        m += string.Format("time elapsed: {0}", dtts);
        m += Environment.NewLine;
        m += string.Format("time remaining: {0}", ddd);
        m += Environment.NewLine;
        m += f;

        textBox1.Text = m;
    }

    //Microsoft.Office.Interop.Word.Document oDoc = oWord.Documents.Open(@"C:\Users\mac\Desktop\2convert\12000termpapers\Educational Studies (224)\acid rain.rtf", oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing);
    //oDoc.SaveAs2(@"C:\Users\mac\Desktop\xxx.txt", Microsoft.Office.Interop.Word.WdSaveFormat.wdFormatText, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing, oMissing);
    //oDoc.Close();
}
```

Для работы необходимо добавить референсы на: `Microsoft.Office.Core` и `Microsoft.Office.Interop.Word`
