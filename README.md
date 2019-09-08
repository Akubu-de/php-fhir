# php-fhir
Tools for creating PHP classes from the HL7 FHIR Specification

# Installation

This library requires the use of [Composer](https://getcomposer.org/)

# Quick start

A convenient download and generation script is included in this repository. 
The script will download current major versions of FHIR into the `input` folder and 
generate classes for every version in the `output` folder.

* Run `composer install`
* Run `php ./bin/generate.php`

```php
Downloading DSTU1 from http://hl7.org/fhir/DSTU1/fhir-all-xsd.zip
Generating DSTU1
Downloading DSTU2 from http://hl7.org/fhir/DSTU2/fhir-all-xsd.zip
Generating DSTU2
Downloading STU3 from http://hl7.org/fhir/STU3/fhir-all-xsd.zip
Generating STU3
Downloading Build from http://build.fhir.org/fhir-all-xsd.zip
Generating Build
Done
```

# Manual Class Generation

## Include library in composer.json

Require entry:
```json
{
  "require": {
    "dcarbone/php-fhir": "dev-master"  
  }
}
```

## Basic Workflow

The first step is to determine the version of the FHIR spec your implementation supports.  Once done, download
the appropriate class definition XSDs from [http://hl7.org/fhir/directory.html](http://hl7.org/fhir/directory.html).

Uncompress the XSD's and place them in a directory that is readable by PHP's runtime user.

Next comes the fun:

## Class Generation

The class generator utility included with this library is designed to parse the XSD's provided by the FHIR
group into PHP classes, complete with markup and type hinting.

There are 2 important things to note with this section:

1. Your exact implementation will probably vary, don't hesitate to ask if you need some help
2. The class generation should be run ONCE per FHIR version.  Once the classes
have been generated they should only ever be re-generated if your server switches to a new FHIR spec

### Example:

```php
require __DIR__.'/vendor/autoload.php';

$schemaPath = 'schema/path';

// first, build new configuration class
$config = new \DCarbone\PHPFHIR\Config([
    // The path to look look for and optionally download source XSD files to
    'schemaPath'  => __DIR__ . '/../input',

    // The path to place generated type class files
    'classesPath' => '__DIR__ . '/../output',

    // If true, will use a noop null logger
    'silent'      => false,

    // If true, will skip generation of test classes
    'skipTests'   => false,

    // Map of versions and configurations to generate
    // Each entry in this map will grab the latest revision of that particular version.  If you wish to use a specific
    // version, please see http://www.hl7.org/fhir/directory.cfml
    'versions'    => [
        'DSTU1' => [
            // Source URL
            'url'       => 'http://hl7.org/fhir/DSTU1/fhir-all-xsd.zip',
            // Namespace to prefix the generated classes with
            'namespace' => '\\HL7\\FHIR\\DSTU1',
        ],
        'DSTU2' => [
            'url'       => 'http://hl7.org/fhir/DSTU2/fhir-all-xsd.zip',
            'namespace' => '\\HL7\\FHIR\\DSTU2',
        ],
        'STU3'  => [
            'url'       => 'http://hl7.org/fhir/STU3/definitions.xml.zip',
            'namespace' => '\\HL7\\FHIR\\STU3',
        ],
        'R4'    => [
            'url'       => 'http://www.hl7.org/fhir/fhir-codegen-xsd.zip',
            'namespace' => '\\HL7\\FHIR\\R4',
        ],
        'Build' => [
            'url'       => 'http://build.fhir.org/fhir-all-xsd.zip',
            'namespace' => '\\HL7\\FHIR\\Build',
        ],
    ],
]);

// next, build definition class
$definition = new \DCarbone\PHPFHIR\Definition($config);
$definition->buildDefinition();

$builder = new \DCarbone\PHPFHIR\Builder($config, $definition);
$builder->build();
```

Using the above code will generate class files under the included [output](./output) directory, under the namespace
` HL7\\FHIR\\{version} `

If you wish the files to be placed under a different directory, pass the path in as the 2nd argument in the
generator constructor.

If you wish the classes to have a different base namespace, pass the desired NS name in as the 3rd argument in the
generator constructor.

## Data Querying

There are a plethora of good HTTP clients you can use to get data out of a FHIR server, so I leave that up to you.

## Response Parsing

As part of the class generation above, a response parsing class called `PHPFHIRResponseParser` will be created
and added into the root namespace directory.  It currently supports JSON and XML response types.

The parser class takes a single optional boolean argument that will determine if it should
attempt to load up the generated Autoloader class.  By default it will do so, but you are free to configure your
own autoloader and not use the generated one if you wish.

### Example:

```php

require 'path to PHPFHIRResponseParserConfig.php';
require 'path to PHPFHIRResponseParser.php';

// build config
$config = new \YourConfiguredNamespace\PHPFHIRResponseParserConfig([
    'registerAutoloader' => true, // use if you are not using Composer
    'sxeArgs' => LIBXML_COMPACT | LIBXML_NSCLEAN // choose different SimpleXML arguments if you want, ymmv.
]);

// build parser
$parser = new \YourConfiguredNamespace\PHPFHIRResponseParser($config);

// provide input, receive output.
$object = $parser->parse($yourResponseData);

```

## JSON Serialization

```php
$json = json_encode($object);
```

## XML Serialization

```php
// To get an instance of \SimpleXMLElement...
$sxe = $object->xmlSerialize();

// to get as XML string...
$xml = $sxe->saveXML();
```

XML Serialization utilizes [SimpleXMLElement](http://php.net/manual/en/class.simplexmlelement.php).

## Testing

As part of class generation, a directory & namespace called `PHPFHIRTests` is created under the root namespace and
output directory.

## TODO

- Implement event or pull-based XML parsing for large responses

## Suggestions and help

If you have some suggestions for how this lib could be made more useful, more applicable, easier to use, etc, please
let me know.
