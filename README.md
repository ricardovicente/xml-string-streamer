xml-string-streamer
===================

What?
-----
Stream very large XML files in PHP with low memory consumption. Is a successor to [XmlStreamer](https://github.com/prewk/XmlStreamer).

Installation
------------

composer.json:
    
    "require": {
        "prewk/xml-string-streamer": "*@dev"
    }

Usage
-----

### 1. Create a stream provider

#### StreamProvider\File

Use this provider to parse large XML files on disk. Pick a chunk size, for example: 16384 bytes.

    $CHUNK_SIZE = 16384;
    $fsp = new \Prewk\XmlStringStreamer\StreamProvider\File("large-xml-file.xml", $CHUNK_SIZE);

#### StreamProvider\Stdout

Not yet available

#### StreamProvider\Guzzle

Not yet available

#### StreamProvider\String

Used only for testing purposes. Streams nothing, just returns a whole string. No chunk closure support.

### 2. Construct the parser and provide the stream provider and a closure

    $parser = new \Prewk\XmlStringStreamer\Parser($fsp, function($xmlString) {
        // This closure will be called every time a full node has been parsed
    });

### 3. Run

    $parser->parse();

Examples
--------

### Simple
Let's say you have a 2 GB XML file __gigantic.xml__ containing customer items that look like this:

    <?xml version="1.0" encoding="UTF-8"?>
    <gigantic>
        <customer>
            <firstName>Jane</firstName>
            <lastName>Doe</lastName>
        </customer>
        ...
    </gigantic>

Parse through it in chunks like this:

    use \Prewk\XmlStringStreamer;
    use \Prewk\XmlStringStreamer\StreamProvider;
    
    // Prepare our stream to be read with a 16kb buffer
    $streamProvider = new StreamProvider\File("gigantic.xml", 16384);

    // Construct the parser and provide a closure to do your stuff
    $parser = new XmlStringStreamer\Parser($streamProvider, function($xmlString) {
        // $xmlString will contain: <customer><firstName>Jane</firstName><lastName>Doe</lastName></customer>
    
        // Load the node with SimpleXML if you want
        $customer = simplexml_load_string($xmlString);
        echo (string)$customer->firstName . "\n";
    });

    // Everything is prepared - time to start parsing
    $parser->parse();

### Handling progress and buffer read callback

You can provide a closure to the stream provider that will fire every time a chunk is read.

    use \Prewk\XmlStringStreamer;
    use \Prewk\XmlStringStreamer\StreamProvider;
    
    // Prepare our stream to be read with a 16kb buffer
    $streamProvider = new StreamProvider\File("gigantic.xml", 16384, function($buffer, $readBytes) {
        // $buffer contains the last read buffer
        // $readBytes equals the total read bytes so far

        echo "Progress: $readBytes / 2 GB\n"; // (Prepare to get spammed)
    });

FAQ
---

### How do I pass stuff to the closure?
    
Check out [Example #3 Closures and scoping](http://www.php.net/manual/en/functions.anonymous.php). Example of the `use` keyword:

    $someCounter = 0;
    $someArray = array();
    $someDbInstance = new SomeDb;

    $parser = new XmlStringStreamer\Parser($streamProvider, function($xmlString) use (&$someCounter, &$someArray, $someDbInstance) {
        $xml = simplexml_load_string($xmlString);

        $counter++;
        $someArray[] = (string)$xml->foo;
        $someDbInstance->query("....");
    });

    $parser->parse();

    echo $counter;
    print_r($someArray);