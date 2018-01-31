---
title: XML学习笔记
date: 2016-07-20 13:31:55
tags: 
- 基础
categories: phper
---
对于XML的学习主要分为两个部分：生成ＸＭＬ和解析ＸＭＬ。

### 生成ＸＭＬ ###
#### 通过字符串生成ＸＭＬ文档 ####
主要有以下几个关键点：
>header('Content-Type:text/xml');
>'<?xml version="1.0"?>'
>htmlspecialchars()

看代码：
```
header('Content-Type: text/xml');
print '<?xml version="1.0"?>' . "\n";
print "<shows>\n";
$shows = array(
		array('name' => 'Modern Family',
			'channel' => 'ABC',
			'start' => '9:00 PM',
			'duration' => '30'),
		array('name' => 'Law & Order: SVU',
			'channel' => 'NBC',
			'start' => '9:00 PM',
			'duration' =>'60')
		);
foreach ($shows as $show) {
	print "<show>\n";
	foreach($show as $tag => $data) {
		print "<$tag>" . htmlspecialchars($data) . "</$tag>\n";
	}
	print "</show>\n";
}

print "</shows>\n";
```
输出的ＸＭＬ文档：
```
<?xml version="1.0"?>
<shows>
	<show>
		<name>Modern Family</name>
		<channel>ABC</channel>
		<start>9:00 PM</start>
		<duration>30</duration>
	</show>
	<show>
		<name>Law &amp; Order: SVU</name>
		<channel>NBC</channel>
		<start>9:00 PM</start>
		<duration>60</duration>
	</show>
</shows>
```
#### 利用ＤＯＭ生成ＸＭＬ文档 ####
关键点：
```
$dom = new DOMDocument('1.0');  
DOMDocument::createElement
DOMNode::appendChild
DOMDocument::createTextNode
DOMDocument::setAttribute
$dom->formatOutput = true;
$books = $dom->saveXML();  //保存为字符串
$dom->save('books.xml');　　//保存为ＸＭＬ文件
```
看代码：
```
$dom = new DOMDocument('1.0');

$book = $dom->appendChild($dom->createElement('book'));
$title = $book->appendChild($dom->createElement('title'));
$title->appendChild($dom->createTextNode('PHP Cookbook'));
$title->setAttribute('edition','3');

$sklar = $book->appendChild($dom->createElement('author'));
$sklar->appendChild($dom->craeteTextNode('Sklar'));

$trachtenberg = $book->appendChild($dom->createElement('author'));
$trachtenberg->appendChild($dom->createTextNode('Trachtenberg'));

// print a nicely formatted version of the DOM document as XML
$dom->formatOutput = true;

echo $dom->saveXML();

<?xml version="1.0"?>
<book>
	<title edition="3">PHP Cookbook</title>
	<author>Sklar</author>
	<author>Trachtenberg</author>
</book>
```
### 解析ＸＭＬ文档 ###
#### 解析简单的ＸＭＬ文档 ####
主要是用到 ` SimpleXML extension`.
>SimpleXML turns elements into object properties.

```
$sx = simplexml_load_file(__DIR__ . '/address-book.xml');
foreach ($sx->person as $person){
	$firstname_text_value = $person->firstname;
	$lastname_text_value = $person->lastname;

	print "$firstname_text_value $lastname_text_value\n";
}
```
#### 解析复杂的ＸＭＬ文档 ####
主要是利用 `DOM extension`.
例如：
```
<books>
	<book>
		<title>PHP Cookbook</title>
		<author>Sklar</author>
		<author>Trachtenberg</author>
		<subject>PHP</subject>
	</book>
	<book>
		<title>Perl Cookbook</title>
		<author>Christiansen</author>
		<author>Torkington</author>
		<subject>Perl</subject>
	</book>
</books>

// find and print all authors
$authors = $dom->getElementsByTagname('author');

// loop through author elements
foreach ($authors as $author) {
	// childNodes holds the author values
	$text_nodes = $author->childNodes;

	foreach ($text_nodes as $text) {
		print $text->nodeValue . "\n";
	}
}

Sklar
Trachtenberg
Christiansen
Torkington
```
#### 解析大型的ＸＭＬ文档 ####
主要使用 `XMLReader extension`.
>There are two major types of XML parsers: ones that hold the entire document in mem‐
ory at once, and ones that hold only a small portion of the document in memory at any
given time.

例如：
```
$reader = new XMLReader();
$reader->open(__DIR__ . '/card-catalog.xml');

/* Loop through document */
while ($reader->read()) {
	/* If you're at an element named 'author' */
	if($reader->nodeType == XMLREADER::ELEMENT &&
$reader->localName == 'author') {
		/* Move to the text node and print it out */
		$reader->read();
		print $reader->value . "\n";
	}
```
#### 使用ＸＰＡＴＨ解析ＸＭＬ ####
主要使用 `XPath`.
>You want to make sophisticated queries of your XML data without parsing the document
node by node.

```
//XPath is available in SimpleXML
$s = simplexml_load_file(__DIR__ . '/address-book.xml');
$emails = $s->xpath('/address-book/person/email');
foreach ($emails as $email) {
	// do something with $email
}

//And in DOM:
$dom = new DOMDocument;
$dom->load(__DIR__ . '/address-book.xml');
$xpath = new DOMXPath($dom);
$emails = $xpath->query('/address-book/person/email');

foreach ($emails as $email) {
	// do something with $email
}
```

至此，ＸＭＬ主要的两部分已完结。

--------------------------------------

下面是关于ＸＭＬ的一些其他方面的知识。

#### Transforming XML with XSLT ####
>You have an XML document and an XSL stylesheet. You want to transform the document
using XSLT and capture the results. This lets you apply stylesheets to your data and
create different versions of your content for different media.

```
// Load XSL template
$xsl = new DOMDocument;
$xsl->load(__DIR__ . '/stylesheet.xsl');

// Create new XSLTProcessor
$xslt = new XSLTProcessor();
// Load stylesheet
$xslt->importStylesheet($xsl);

// Load XML input file
$xml = new DOMDocument;
$xsl->load(__DIR__ . '/address-book.xml');

// Transform to string
$results = $xslt->transformToXML($xml);
$results = $xslt->transformToURI($xml, 'results.txt');

// Transform to DOM object
$results = $xslt->transformToDoc($xml);
```
####  Reading RSS and Atom Feeds ####
>You want to retrieve RSS and Atom feeds and look at the items.
>**Use the MagpieRSS parser. **

```
require __DIR__ . '/magpie/rss_fetch.inc';
$feed = 'http://news.php.net/group.php?group=php.announce&format=rss';
$rss = fetch_rss( $feed );
print "<ul>\n";

foreach ($rss->items as $item) {
	print '<li><a href="' . $item['link'] . '">' . $item['title'] . "</a></li>\n";
}
print "</ul>\n";
```
#### Writing RSS Feeds ####
```
class rss2 extends DOMDocument {
	private $channel;

	public function __construct($title, $link, $description) {
		parent::__construct();
		$this->formatOutput = true;

		$root = $this->appendChild($this->createElement('rss'));
		$root->setAttribute('version', '2.0');
		
		$channel= $root->appendChild($this->createElement('channel'));

		$channel->appendChild($this->createElement('title', $title));
		$channel->appendChild($this->createElement('link', $link));
		$channel->appendChild($this->createElement('description', $description));

		$this->channel = $channel;

	}

	public function addItem($title, $link, $description) {
		$item = $this->createElement('item');
		$item->appendChild($this->createElement('title', $title));
		$item->appendChild($this->createElement('link', $link));
		$item->appendChild($this->createElement('description', $description));

		$this->channel->appendChild($item);
	}
}

$rss = new rss2('Channel Title', 'http://www.example.org', 'Channel Description');

$rss->addItem('Item1', 'http://www.example.org/item1','Item 1 Description');
$rss->addItem('Item2', 'http://www.example.org/item2','Item 2 Description');

print $rss->saveXML();

<?xml version="1.0"?>
<rss version="2.0">
	<channel>
	<title>Channel Title</title>
	<link>http://www.example.org</link>
	<description>Channel Description</description>
	<item>
		<title>Item 1</title>
		<link>http://www.example.org/item1</link>
		<description>Item 1 Description</description>
	</item>
	<item>
		<title>Item 2</title>
		<link>http://www.example.org/item2</link>
		<description>Item 2 Description</description>
	</item>
	</channel>
</rss>
```

