---
layout: post
title: "正则表达式中\\d和[0-9]有什么区别"
date: 2013-06-04
comments: true
tags: [CSharp, Regex]
---
<p>今天看到Stackoverflow上一个有趣的<a href="http://stackoverflow.com/questions/16621738/d-is-less-efficient-than-0-9?newsletter=1&amp;nlcode=55866%7cc739">问题</a>，为什么正则表达式在中\d比[0-0]低效？</p>
<p>提问者用了如下的代码来做测试：</p>


```csharp
static void Main(string[] args)
        {
            var rand = new Random(1234);
            var strings = new List<string>();
            //10K random strings
            for (var i = 0; i < 10000; i++)
            {
                //Generate random string
                var sb = new StringBuilder();
                for (var c = 0; c < 1000; c++)
                {
                    //Add a-z randomly
                    sb.Append((char)('a' + rand.Next(26)));
                }
                //In roughly 50% of them, put a digit
                if (rand.Next(2) == 0)
                {
                    //Replace one character with a digit, 0-9
                    sb[rand.Next(sb.Length)] = (char)('0' + rand.Next(10));
                }
                strings.Add(sb.ToString());
            }

            var baseTime = testPerfomance(strings, @"\d");
            Console.WriteLine();
            var testTime = testPerfomance(strings, "[0-9]");
            Console.WriteLine("  {0:P2} of first", testTime.TotalMilliseconds / baseTime.TotalMilliseconds);
            testTime = testPerfomance(strings, "[0123456789]");
            Console.WriteLine("  {0:P2} of first", testTime.TotalMilliseconds / baseTime.TotalMilliseconds);
        }

        private static TimeSpan testPerfomance(List<string> strings, string regex)
        {
            var sw = new Stopwatch();

            int successes = 0;

            var rex = new Regex(regex);

            sw.Start();
            foreach (var str in strings)
            {
                if (rex.Match(str).Success)
                {
                    successes++;
                }
            }
            sw.Stop();

            Console.Write("Regex {0,-12} took {1} result: {2}/{3}", regex, sw.Elapsed, successes, strings.Count);

            return sw.Elapsed;
        }
    }
```
<p>得到的输出结果是：</p>


```
Regular expression \d           took 00:00:00.2141226 result: 5077/10000
Regular expression [0-9]        took 00:00:00.1357972 result: 5077/10000  63.42 % of first
Regular expression [0123456789] took 00:00:00.1388997 result: 5077/10000  64.87 % of first
```

<p>从这个测试中可以看出\d比[0-9]慢了一倍。</p>
<h3>&nbsp;</h3>
<p>原因在于，\d会比较所有的unicode的数字，包括</p>

```
0123456789٠١٢٣٤٥٦٧٨٩۰۱۲۳۴۵۶۷۸۹߀߁߂߃߄߅߆߇߈߉०१२३४५६७८९০১২৩৪৫৬৭৮৯੦੧੨੩੪੫੬੭੮੯૦૧૨૩૪૫૬૭૮૯୦୧୨୩୪୫୬୭୮୯௦௧௨௩௪௫௬௭௮௯౦౧౨౩౪౫౬౭౮౯೦೧೨೩೪೫೬೭೮೯൦൧൨൩൪൫൬൭൮൯๐๑๒๓๔๕๖๗๘๙໐໑໒໓໔໕໖໗໘໙༠༡༢༣༤༥༦༧༨༩၀၁၂၃၄၅၆၇၈၉႐႑႒႓႔႕႖႗႘႙០១២៣៤៥៦៧៨៩᠐᠑᠒᠓᠔᠕᠖᠗᠘᠙᥆᥇᥈᥉᥊᥋᥌᥍᥎᥏᧐᧑᧒᧓᧔᧕᧖᧗᧘᧙᭐᭑᭒᭓᭔᭕᭖᭗᭘᭙᮰᮱᮲᮳᮴᮵᮶᮷᮸᮹᱀᱁᱂᱃᱄᱅᱆᱇᱈᱉᱐᱑᱒᱓᱔᱕᱖᱗᱘᱙꘠꘡꘢꘣꘤꘥꘦꘧꘨꘩꣐꣑꣒꣓꣔꣕꣖꣗꣘꣙꤀꤁꤂꤃꤄꤅꤆꤇꤈꤉꩐꩑꩒꩓꩔꩕꩖꩗꩘꩙０１２３４５６７８９
```

<p>可以从<a href="http://www.fileformat.info/info/unicode/category/Nd/list.htm">这里</a>看到更全的列表，列出了所有Unicode中属于数字的字符。</p>
<h3>&nbsp;</h3>
<p>如果在生成Regex的时候传入一个参数<code>RegexOptions.ECMAScript</code>，如下所示，那么\d就和[0-9]的效率一样了。可以从<a href="http://msdn.microsoft.com/en-us/library/yd1hzczs.aspx">这里</a>找到更多的Regex的选项。</p>

```csharp
var rex = new Regex(regex, RegexOptions.ECMAScript);
```