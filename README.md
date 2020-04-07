


# Kobra Framework for Wordpress #

Kobra framework is a simple model 2 like framework, for developing plugins and themes for Wordpress 5+.

## Table of contents ##

- [Kobra Framework for Wordpress](#kobra-framework-for-wordpress)
  * [Table of contents](#table-of-contents)
  * [Requiremets](#requiremets)
  * [Project structure](#project-structure)
  * [Plugin structure](#plugin-structure)
  * [Using in your plugin](#using-in-your-plugin)
    + [Create kobrafw.xml](#create-kobrafwxml)
    + [Init](#init)
    + [The Controller objects](#the-controller-objects)
    + [The Service objects](#the-service-objects)
    + [The Dao objects](#the-dao-objects)
    + [The Entity objects](#the-entity-objects)
    + [The Dto objects](#the-dto-objects)
  * [Using in Theme](#using-in-theme)
    + [basic structure](#basic-structure)
    + [Init](#init-1)
    + [theme.yml](#themeyml)
  * [Using Helpers](#using-helpers)
  * [Create Gutenberg block](#create-gutenberg-block)
    + [Create block controllers](#create-block-controllers)
    + [Service and Helper](#service-and-helper)
  * [Use of Rest\Ajax](#use-of-rest-ajax)
    + [Backend:  your rest service](#backend---your-rest-service)
      - [The REST route](#the-rest-route)
      - [The controller](#the-controller)
      - [The service (ajax)](#the-service--ajax-)
      - [KobrafwWrapAjax status](#kobrafwwrapajax-status)
    + [Frontend](#frontend)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## Requiremets ##

* PHP 7.2+
* Wordpress 5+
* cURL

## Project structure ##

## Plugin structure ##
```
kobrafw
|
+-- blocks
|   +-- ctrl
|   |   +-- your-guntenberg-block-ctrl.class.php       
|   +-- dto
|   |   +-- your-guntenberg-block-dto.class.php     
|   +-- helpers
|   |   +-- your-guntenberg-helper.class.php    
|   +-- js
|   |   +-- block-name
|   |   |   +-- gutemberg-code-build.js    
|   |   |   +-- gutemberg-code-editor.css  
|   +-- service
|   |   +-- your-guntenberg-block-service.class.php     
+-- ctrl
|   +-- your-class-ctrl.class.php
+-- dao
|   +-- your-dao.class.php     
+-- dto
|   +-- your-dto.class.php  
+-- entities
|   +-- your-entities.class.php  
+-- exceptions
|   +-- your-exception.class.php  
+-- helpers
|   +-- your-helper.class.php   
+-- services
|   +-- your-service.class.php  
+-- templates
|   +-- your-template.class.php   
+-- view
|   +-- your-view.class.php   
|
+-- kobrafw.xml
```

## Using in your plugin ##

The plugin structure:

```
YOUR PLUGIN
|
+-- kobrafw
|   +-- (package)service_name
|   +-- service_name.php
+-- index.php
```

* **package_service_name**: Root package
* **service_name.php**: The bootstrap file
* **index.php**: plugin definition file


**Important note**:
The base file name of the bootstrap file (ex. service_name.php  => service_name)  must match the name of the root package.

Define your plugin in index.php

```php
/*
 * Plugin Name: Kobra framework sample plugin
 ...
```

Include your service/module:

```php
include 'kobrafw/kobrasample.php';
```

Your __service_name.php__  add the following line for bootstrapping the plugin (load all classes through autoloader) :

```php
use Kobrafw\KobrafwPlugin;

/** attvazione kobra plugin */
add_action( 'init', function() { new KobrafwPlugin( __FILE__ ); });

```


### Create kobrafw.xml ###
Create your kobrafw.xml in `package_service_name` directory.
Define the basic file structure 
```
<kobrafw>

	<prefix>kf_</prefix>

	<package>kobrasample</package>

</kobrafw>
```
### Init ###
In your functions.php add this lines to instantiate KobrafwTheme 

```
use Kobrafw\KobrafwPlugin;

/** kobra plugin bootstrap */
add_action( 'init', function() { new KobrafwPlugin( __FILE__, __DIR__ ); });

```

### The Controller objects ###
The __controller__ object accepts input and converts it to commands for services.

### The Service objects ###


The __service__ object is responsibile of logical part of the application. It alse bind __DTO__ object to __Entity__ and then passes them to __DAO__

The service must extend **KobrafwServiceMain** 

```php
class YourclassService extends KobrafwServiceMain {
```

In _service_ you can:
* write business logic
* at the same way as normal service you ca use any other object type DTO, DAO, Entity.
* use automatic bind of form into DTO object.

Inside controller in the method declared as callback:

```php
public function yourCallback($request) {
```

you can use the previously "injected" _Service_ object

```xml
...
     <service>
     	   <classname>Yourpackage\services\YourclassService</classname>
     </service>     
....
```

**Example** 


You can create a function in your service

```php
public function sayHello() {
 return "hello";
}
```

and then in your callback function of Controller your can call the service in this way:

```php
$hello = $this->getService()->sayHello();
```


### The Dao objects ###
The __DAO__ is responsible for managing the persisted data, it receive Entity object from service.

### The Entity objects ###
The entity maps the database table.

### The Dto objects ###
The __DTO__  (data transfer object) is an object that carries data beetwen front end and Service.
The __DTO__ must extends __KobrafwDtoMain__ and inherits methods :
* bindForm: this method bind the parameter from `$_REQUEST__` array to object property matching the name
* bind:  this method bind the parameter from property of __Entity__ object to __DTO__ property matching the name


### Custom Meta Boxes ###
####  Simple metabox ####
To create a meta box your kobrafw.xml define a new box with "metabox" type.

Attribute:

 - name = the name of the block
 - type  = __metabox__

Elements:

 - id: the id of metabox (required)
 - title: the title of the metabox (required)
 - post-type: the post type will appear the metabox(_posts_, _product_, _etc.-)
 - fields: the list of field :
	 - id: the identify of field
	 - label. the label of field

```xml
<kobrafwobj name="metabox-products" type="metabox,global">
		<ctrl>
			<classname>Yourpackage\ctrl\YourMetaboxCtrl</classname>
		</ctrl>
		<id>cocktail-metabox</id>
		<title>Cocktail</title>
		<post-type>product</post-type>
		<fields>
			<field>
				<id>cocktail</id>
				<label>wp_editor</label>
			</field>
		</fields>
	</kobrafwobj>
```
##### The controller ####
In this case controller can be empty
```php

namespace Yourpackage\ctrl;

use Kobrafw\ctrl\KobrafwCtrlMetabox;
if ( ! defined( 'ABSPATH' ) ) {
	exit();

}

class YourMetaboxCtrl extends KobrafwCtrlMetabox {
}
```

####  Advanced metabox ####
You can create a metabox with particular field type, such as Select TinyMCE editor using a custom callback

```xml
<kobrafwobj name="metabox-products" type="metabox,global">
		<ctrl>
			<classname>Yourpackage\ctrl\YourMetaboxCtrl</classname>
		</ctrl>
		<id>cocktail-metabox</id>
		<title>Cocktail</title>
		<post-type>product</post-type>
		<callback>cocktailMetabox</callback>
		<fields>
			<field>
				<id>cocktail</id>
				<label>wp_editor</label>
			</field>
		</fields>
	</kobrafwobj>
```

##### The controller ####
In this case controller must implement the callback previously declared
Example:
```php

namespace Yourpackage\ctrl;

use Kobrafw\ctrl\KobrafwCtrlMetabox;
if ( ! defined( 'ABSPATH' ) ) {
	exit();

}

class YourMetaboxCtrl extends KobrafwCtrlMetabox {

	public function cocktailMetabox($post) {
			$text= get_post_meta($post->ID, 'cocktail' , true );
			wp_editor( htmlspecialchars_decode($text), 'cocktail', $settings = array('textarea_name'=>'cocktail') );
	}

}
```


## Using in Theme ##

### basic structure ###
```
YOUR THEME
|
+-- hooks
|
+-- styles
|   +-- your-style.css
+-- scripts
|   +-- your-scripts.php  
```

### Init ###
In your functions.php add this lines to instantiate KobrafwTheme 

```
use Kobrafw\KobrafwTheme;

add_action( 'init', 'YOUR_THEME_INIT_FUNC' );

function YOUR_THEME_INIT_FUNC() {
	new KobrafwTheme( wp_get_theme() );
}

```
### theme.yml ###
The file theme.yml allow to register scripts and css

```
stylesheets:
   - css: 
       name: normalize
       src: normalize.css
       version: 1 
scripts:
   - script:
        name: jquery
        src: https://d3e54v103j8qbb.cloudfront.net/js/jquery-3.4.1.min.220afd743d.js
        version: 1
        in_footer: true
```

The base folder for stylesheets is "styles" and for scripts is "scripts", then if you not use remote resources you must to create  anche place your files in "styles" and "scripts" folder.   

### Theme helpers ###

#### Custom Action Hooks ####
To create a custom action hook don't use functions.php file in your theme, but place your hook in separate files (in every files group similar functions) inside ```hooks``` folder inside your theme.



#### Theme customizers ####
To add to your custom theme settings using wordpress customizer ([https://wordpress.com/support/customizer/](https://wordpress.com/support/customizer/)), you can use  ```KobrafwHelperThemeCustomize``` in your theme functions.php.

 - in `hooks` folder create file `customizers.php`
 - in file `customizers.php` refer  ```KobrafwHelperThemeCustomize```
 - create an array with settings, every fields in associative array has the followings key:
	 - "id" =>  //identify,
	 * "settings" =>  \\field es. phone_number
	 * "section" =>  \\section name in menu
	 * "label" =>   \\field label
	 * "type" =>  \\field type: only text, email and URL available
	 * "default_value" >  \\ "the placeholder"
	 * "priority" =>  \\ default value 30 
* create the JavaScript File in theme's `scripts` folder
* call `createCustomizer` with `id section`, `name section`, `settings`, `js file` parameters

Example
```php
use Kobrafw\helpers\KobrafwHelperThemeCustomize;

$fields = array();

$text_domain = "sample-theme";

$field = array(
	"id" => "id_phone_number",
	"settings" => "phone_number",
	"section" => "sample-theme_params",
	"label" => __( 'Telephone number', $text_domain ),
	"type" => "text",
	"default_value" => "",
	"priority" => 30
);
array_push($fields, $field);

KobrafwHelperThemeCustomize::createCustomizer("origine_params", "My Settings", $fields, "customizer.js");
```  



## Logging ##
To log in you defined file or into php error handling routines.




## Create custom Helpers ##
Helper are classes with static method only.

To declare helper add the followiing lines to kobrafw.xml

```xml
<helper>
		<name>your-helper-name</name>
		<classname>Package\helper\ExampleHelper
		</classname>
</helper>
```

## Create Wordpress page ##


## Create Web Components (Reactjs) ##

To create webcomponents for your plugin declare a new __kobrafwobj__ elements with the __type__ property set to "__webcomponent__".

In the js __elements__ there are the following properties:

__JS__
- **name**: the name of the web component
- **filename**: the path of your js file from root plugin folder
- **dep**: dependency of other script (name of the script)
- **version**: the version
- **in_footer**: 1 if you want to place the script in footer, 0 in head

__CSS__
- **name**: the name of the script
- **filename**: the path of your css file from root plugin folder
  


 - List item

```xml
	<kobrafwobj type="webcomponent" name="contact-form">

		<js>
			<name>contact-form</name>
			<filename>folder/scripts/webcomponents/example/public/bundle.js</filename>
			<dep>wp-element</dep>
			<version>0.1</version>
			<in_footer>1</in_footer>			
		</js>
		
    	<css>
			<name>contact-form</name>
			<filename>greenpower/styles/contact-form.css</filename>
		</css>

	</kobrafwobj>
```




## Create Gutenberg block ##
To create blocks you must re-create a folder structure similar to the following, under _blocks_ folder:

```
+-- blocks
|   +-- ctrl
|   |   +-- your-guntenberg-block-ctrl.class.php       
|   +-- dto
|   |   +-- your-guntenberg-block-dto.class.php     
|   +-- helpers
|   |   +-- your-guntenberg-helper.class.php    
|   +-- js
|   |   +-- block-name
|   |   |   +-- gutemberg-code-build.js    
|   |   |   +-- gutemberg-code-editor.css  
|   +-- service
|   |   +-- your-guntenberg-block-service.class.php 

```



### Create block controllers ###
The controller must extend KobrafwCtrlBlock

```php
class SampleBlocksCtrl extends KobrafwCtrlBlock {
```

### Service and Helper ###
You can use __Service__ and __Helper__ as kobra standard:

 - [service object](#the-service-objects)
 - [helpers object](#using-helpers)


### Blocks in the kobrafw.xml ###

To activate the block it is necessary create define the node *kobrafwobj* with attribute *type* with "editor-block" value.

```xml
	<kobrafwobj name="example-blocks"
		type="editor-block,global">
```
**Note**: the value _global_ is necessary in case you need to use the controllers or sever side rendering.

The properties (child nodes) available are:

 - dir: the folder where the bloks in JS where placed
 - version: the scripts version
 - ctrl: the controllers
 - service: the services
 - block:  the name of the block, the name must be block.js 

Example:
```xml
	<kobrafwobj name="example-blocks"
		type="editor-block,global">
		
		<dir>blocks/scripts/block-sample</dir>

		<version>1</version>

		<ctrl>
			<classname>Mypackage\blocks\ctrl\SampleBlocksCtrl</classname>
		</ctrl>
		<service>
			<classname>Mypackage\blocks\services\SampleBlocksService</classname>
		</service>

		<blocks>
			<block>event-sample</block>
			<block type="server-side" callback="serverSide">plan-sample</block>
		</blocks>

	</kobrafwobj>

```


### Create a Reactjs Block ###

#### Install npm modules ####
Go in the plugin's  _package_  folder and then in the _blocks/scripts_.
Initialize npm block with
```bash
npm init @wordpress/block _[YOUR BLOCK FOLDER]_ 
```
Start compiling with:
```bash
npm start
```
Your source file __index.js__ is placed in src folder.

#### Create  your custom block ####
Create a block  as explained on ["Writing Your First Block Type"](https://developer.wordpress.org/block-editor/tutorials/block-tutorial/writing-your-first-block-type/).

In __registerBlockType__  add the block name using the _kobrafwobj_ __name__ attribute value and the name of the block  declared in the __\<block\>\</block\>__

```javascript
registerBlockType('example-blocks/event-sample', {
```

Example
```javascript

const { registerBlockType } = wp.blocks;

registerBlockType('example-blocks/event-sample', {
	title: __("Event Sample"), 
	icon: "megaphone", 
	category: "common", 
	edit: function  edit(props) {
		return  <div  class="scheduleWrapper">hello</div>;
	},
	save: function(props) {
		return  <div  class="scheduleWrapper">hello</div>;
	}
});
```

### Server side rendering (dynamic blocks) ###

In order to create a server side block for gutenberg you need to define an element __block__ in kobrafw.xml, with
attribute type _"server-side"_ .
The attribute __callback__ is the name of the method in previously declared controller that 
 
```xml
<block type="server-side" callback="serverSide">test-serverside</block>			
```

#### The src/index.js ####

Install the npm module for server side renderas explained on [@wordpress/server-side-render](https://developer.wordpress.org/block-editor/packages/packages-server-side-render/)

Create a dynamic block  as explained on ["Creating dynamic blocks"](https://developer.wordpress.org/block-editor/tutorials/block-tutorial/creating-dynamic-blocks/)), in the paragraph _Live rendering in the block editor_

In __registerBlockType__  add the block name using the _kobrafwobj_ __name__ attribute value and the name of the block  declared in the __\<block\>\</block\>__

```javascript
registerBlockType('example-blocks/test-serverside', {
```

Example:
```javascript
const { registerBlockType } = wp.blocks;
const { ServerSideRender } = wp;

registerBlockType( 'example-blocks/test-serverside', {
	title: 'Example: Server side',
    icon: 'megaphone',
    category: 'common',
	edit: function( props ) {
		return (
			<ServerSideRender block="example-blocks/test-serverside"	/>
		);
	},
	save: function(props) {
		//server side rendering
		return  null;
	}
} );
```
#### The callback ####
Inside the controller you need to create the "serverSide" method (as the attribute __callback__ in _kobrafw.xml_). You can instantiate _service_ and _view_ and use the helper __KobrafwHelperMinify__ to minify html output

Example:
```php
	public function serverSide($attributes, $content) {
		return  KobrafwHelperMinify::minifyHTML("<div>Hello!</div>");
	}

```

## (API) Credentials ##
It is possible to store in _kobrafw.xml_ a pairs of access keys associated with a host in which to use it. The key will be provided by calling a REST endpoint.

**Endpoint**
```
[SITE URL]/wp-json/kobrafw/v1/auth/[name of service]?encode=base64
```
or with not encoded outpout
```
[SITE URL]/wp-json/kobrafw/v1/auth/[name of service]?encode=base64
```
The entry in _kobrafw.xml_
```xml

<credentials>
		<service>
			<name>woocommerce</name>
			<key>ck_jlk089278489832jh24872384y824</key>
			<secret>cs_lksjdf89sud8fs8f8sdf8asda3</secret>
			<host>www.example.com</host>
		</service>
		<service>
			<name>woocommerce</name>
			<key>ck_jlk089278489832jh24872384y824</key>
			<secret>cs_lksjdf89sud8fs8f8sdf8asda3</secret>
			<host>stage.example.com</host>
		</service>
	</credentials>
	
```

**Example output**
```json
{
"sussess": true,
"data": [
	{
	"key": "ck_8b54f71916c197984c21f91925988c202b6f5831",
	"secret": "cs_69189fa2b4869ac1cc617fcedaf7ed27d21cb4e7",
	"bundle": "ck_8b54f71916c197984c21f91925988c202b6f5831:cs_69189fa2b4869ac1cc617fcedaf7ed27d21cb4e7"
	},
],
"message": null
}
```


## Use of Rest\Ajax ##

### Backend:  your rest service ###

* First you have to create your own _Controller_ and _Service_. After, if you need, you can create DTO, DAO and Entity objects


```
kobrafw
|
+-- ctrl
|   +-- your-class-ctrl.class.php
+-- services
|   +-- your-service.class.php  
```

* Activation of backend service: configuration in _kobrafw.xml_

```xml
<kobrafw>
<rest_api_version>1</rest_api_version>
<package>my-app</package>

...

<kobrafwobj type="ajax,global" name="your-rest-service-name">
    <actions>
        <!-- route name -->
    	    <route>your-route-name</route>		
        <!-- ref to see possibile method: https://developer.wordpress.org/reference/classes/wp_rest_server -->
        <method>READABLE</method>
        <!-- method in your Controller -->        
        <callback>yourCallback</callback>
        <!-- 
         permissions method in your Controller 
         look at https://developer.wordpress.org/rest-api/extending-the-rest-api/adding-custom-endpoints/#permissions-callback
        -->                
        <permissions>yourCallbackPermission</permissions>			
    </actions>
	        
     <ctrl>
     	   <classname>Yourpackage\ctrl\YourclassCtrl</classname>
     </ctrl>
     <service>
     	   <classname>Yourpackage\services\YourclassService</classname>
     </service>     
     ....
```

Then you can associate to each action/route, a method in the Controller. In this way you can use only one Controller for many route 

#### The REST route ####
the route will then be composed as follows:

```
<SITE URL: https://www.yoursite.com">/wp-json/<package>/v<rest_api_version>/<kobrafwobj name>/<route>

```
Example:

```
https://www.yoursite.com/wp-json/my-app/v1/your-rest-service-name/your-route-name
```



#### The controller ####

The ajax controller must extend Kobrafw_Ctrl_Ajax

```php
class YourclassCtrl extends KobrafwCtrlAjax {
```

Your controller must extend KobrafwCtrlAjax

```php
public function yourCallback($request) {
```

Verification of the nonce

```php
parent::checkNonce($params->api_nonce);
```

Use **KobrafwWrapAjax** method to format json output. 

 - **setMessage**: set a plain text message
 - **setList**: set an array of results
 - **setDto**: set an object of result

```php
$kobra_ajax = new KobrafwWrapAjax();

try {
	$params = (object) $request->get_params();
	parent::checkNonce($params->api_nonce);
	$kobra_ajax->setStatus(200);
	$kobra_ajax->setMessage("hello");
} catch (Exception $e) {
	$kobra_ajax->setStatus(500);
	$kobra_ajax->setMessage($e->getMessage());
}
$kobra_ajax->response();
exit;

```


#### KobrafwWrapAjax status ####

* 200 => '200 OK'
* 400 => '400 Bad Request',
* 422 => 'Unprocessable Entity',
* 500 => '500 Internal Server Error'

#### The service object (ajax) ####

The service must extend **KobrafwServiceMain**, as standard service

See   [The service object](#the-service-objects)


### Frontend ###

* API REST URL: kobra_ajax.api_url
* NONCE: kobra_ajax.api_nonce

Example of JSON response

```json
{ 
   "sussess":true,
   "data":{ 
      "name":"The name",
      "email":"email@email.com"
   },
   "message":"Email sent"
}
```

Example of ajax request made in jQuery

```javascript

$.ajax({
          method: 'POST',
          url: kobra_ajax.api_url, 
          data: data,
          beforeSend: function ( xhr ) {
              xhr.setRequestHeader( 'X-WP-Nonce', kobra_ajax.api_nonce );
          },
          success : function( response ) {
            console.log(response);
            $( '#result' ).html(response.message);
          },
          fail : function( response ) {
            console.log(response);
            $( '#result' ).html(response.message);
          }
      });

```

