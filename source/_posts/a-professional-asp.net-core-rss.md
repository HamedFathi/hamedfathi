---
title: A Professional ASP.NET Core - RSS
date: October 9 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - rss
---

`RSS` feeds provide an excellent mechanism for websites to publish their content for consumption by others.

<!-- more -->

Install the below package

```bash
Install-Package System.ServiceModel.Syndication -Version 4.7.0
dotnet add package System.ServiceModel.Syndication --version 4.7.0
<PackageReference Include="System.ServiceModel.Syndication" Version="4.7.0" />
```

## Create the model

```cs
public class RssPost
{
    public string Title { get; set; }
    public string UrlSlug { get; set; }
    public string Preview { get; set; }
    public DateTime PostDate { get; set; }
}
```

## Create the action

Create the action that will respond to the request for our RSS Feed.

```cs
using System.ServiceModel.Syndication;
using System.Xml;
using System.IO;
using System.Text;

[ResponseCache(Duration = 1200)]
[HttpGet]
public IActionResult Rss()
{
    var feed = new SyndicationFeed("Title", "Description", new Uri("https://hamedfathi.me"), "RSSUrl", DateTime.Now);

    feed.Copyright = new TextSyndicationContent($"{DateTime.Now.Year} Hamed Fathi");
    var items = new List<SyndicationItem>();
    var postings = new List<RssPost>();
    foreach (var item in postings)
    {
        var postUrl = Url.Action("Article", "Blog", new { id = item.UrlSlug }, HttpContext.Request.Scheme);
        var title = item.Title;
        var description = item.Preview;                
        items.Add(new SyndicationItem(title, description, new Uri(postUrl), item.UrlSlug, item.PostDate));
    }

    feed.Items = items;
    var settings = new XmlWriterSettings
    {
        Encoding = Encoding.UTF8,
        NewLineHandling = NewLineHandling.Entitize,
        NewLineOnAttributes = true,
        Indent = true
    };
    using (var stream = new MemoryStream())
    {
        using (var xmlWriter = XmlWriter.Create(stream, settings))
        {
            var rssFormatter = new Rss20FeedFormatter(feed, false);
            rssFormatter.WriteTo(xmlWriter);
            xmlWriter.Flush();
        }
        return File(stream.ToArray(), "application/rss+xml; charset=utf-8");
    }
}
```

The only important note here is that I am setting a `ResponeCache` attribute on this method. I strongly recommend this as your `RSS` Feed generation is often something that will be going to the database etc. and feed-readers are notorious for multiple refreshes etc. By enabling ResponseCaching for your `RSS` action, you can reduce the load on your server.

## Reference(s)

Most of the information in this article has gathered from various references.

* https://mitchelsellers.com/blog/article/creating-an-rss-feed-in-asp-net-core-3-0
* https://khalidabuhakmeh.com/reading-rss-feeds-with-dotnet-core
* http://www.binaryintellect.net/articles/05fc3052-bf5c-4ab9-b8ab-a7fd6974b977.aspx