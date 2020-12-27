# XpressEngine(Rhymix)_sitemap_python project
This repo include Python based - CMS `XpressEngine(XE)` or `Rhymix` sitemap XML generation code.
- Connect to MySQL DB
- Run query to fetch `XE` documents metadata
- Fix query result to [sitemap.xml format](https://www.sitemaps.org/ko/protocol.html)

# Getting Started
TODO: Setup python 3.6+ environment and run notebook code
1. Install mysql client, dotenv
2. Clone this repo.
3. Setup your `.env` file
4. Open `sitemap.ipynb` notebook file
5. Change SQL query to fit your XpressEngine DB in notebook file
6. Run notebook file
7. Automation run, make py file and add task in OS scheduler(crontab or Windows Server scheduler)

## XE - MySQL query
- Check your module `mid` name to fetch it.
- Make rule for sitemap XML data
- `priority`, `changefreq` by XE regdate compare
- build `loc` URL with concat with `mid` &  `document_srl`
- In admin page, list up `mid` board or page to add to sitemap, then add to query
- Maximum sitemap count limit is 50,000 

```sql
select 
rx3_documents.document_srl, 
rx3_documents.module_srl, 
rx3_modules.module, 
left(rx3_documents.regdate, 8),
rx3_modules.mid,
cast(concat('https://www.<YOUR-DOMAIN>.com/', mid, '/', document_srl) as CHAR(10000) CHARACTER SET utf8) as loc,
datediff(curdate(), rx3_documents.regdate) as dd,
CASE
    WHEN datediff(curdate(), rx3_documents.regdate) <= 7 THEN 'daily'
    WHEN datediff(curdate(), rx3_documents.regdate) > 7 and datediff(curdate(), rx3_documents.regdate) <= 30 THEN 'weekly'
    WHEN datediff(curdate(), rx3_documents.regdate) > 30 and datediff(curdate(), rx3_documents.regdate) <= 365 THEN 'monthly'
    WHEN datediff(curdate(), rx3_documents.regdate) > 365 THEN 'yearly'
END as changefreq,
CASE
    WHEN datediff(curdate(), rx3_documents.regdate) <= 7 THEN 0.8
    WHEN datediff(curdate(), rx3_documents.regdate) > 7 and datediff(curdate(), rx3_documents.regdate) <= 30 THEN 0.7
    WHEN datediff(curdate(), rx3_documents.regdate) > 30 and datediff(curdate(), rx3_documents.regdate) <= 365 THEN 0.6
    WHEN datediff(curdate(), rx3_documents.regdate) > 365 THEN 0.5
END as priority
from rx3_documents
    inner join rx3_modules 
        on rx3_documents.module_srl = rx3_modules.module_srl
where 
rx3_modules.module like 'page'
or mid in ('board_FreeTalk', 'board_Photo', 'board_Local')  # module mid list
order by rx3_documents.regdate desc
limit 49500;
```

## sitemap generation python code
Simply generate XML and save it as UTF-8 encoding.

```python
import xml.etree.cElementTree as ET

d = {'xmlns':'http://www.sitemaps.org/schemas/sitemap/0.9'}  # root dict
root = ET.Element("urlset", d)

for tup1 in res:
    url = ET.SubElement(root, "url")
    loc = tup1[5]
    lastmod = set_date(tup1[3]) 
    changefreq = tup1[7]
    priority = str(tup1[8])
    ET.SubElement(url, "loc").text = loc
    ET.SubElement(url, "lastmod").text = lastmod
    ET.SubElement(url, "changefreq").text = changefreq
    ET.SubElement(url, "priority").text = priority

tree = ET.ElementTree(root)

# set ET to str
str_xml = ET.tostring(root, encoding="utf-8", xml_declaration=True).decode('utf-8')

# write to siteml.xml file
text_file = open("sitemap.xml", "w", encoding="utf-8")
text_file.write(str_xml)
text_file.close()
```

# Build and Test
- Compatible with Python 3.6+

# Contributor
- CloudbreadPaPa