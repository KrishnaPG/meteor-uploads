In this post I'll introduce two new packages using which you can easily setup a file upload service to your Meteor server. This solution has following perks:

1. Uses the famous [jQuery file upload](https://blueimp.github.io/jQuery-File-Upload/) system from blueimp. 
	1. As a result you can upload any file size (default limit 11GB)
	2. **Displays upload progress**
	3. **Uploads can be canceled**
	4. Images can be resized to various sizes using the *imagemagick* 
1. Saves and serves file from arbitrary folder on your server. This solves problems with hot-code reload, when files are saved into Meteor's *public* directory
	1. Possibility to save files to subfolders
	2. Possibility to rename files on the server
1. Simple configuration and installation! No need to install 10+ packages like with Collection-FS solution 

```
Please note that since we are using blueimp's juery solution, this solution has a full potential of performing client image resizes, chunked uploads, upload resume and more. These features will be gradually added. Pull requests are welcome!
```

Demo application can be found on [Github]()

# Quick Start

Install packages
```
$ meteor add tomi:upload-server
$ meteor add tomi:upload-jquery
```

Create directory in the application root
```
mkdir -p .uploads/tmp
```

Configure server

```javascript
//file:/server/init.js
Meteor.startup(function () {
  UploadServer.init({
    tmpDir: process.env.PWD + '/.uploads/tmp',
    uploadDir: process.env.PWD + '/.uploads/'
  })
});
```
	
Use template

```html
<template name="yourTemplate">
	{{> upload_sematic_ui }}
</template>
```

DONE!

# Installation

The installation is very simple and consists of installing two packages:

```bash
$ meteor add tomi:upload-server
$ meteor add tomi:upload-jquery
```

I have separated these two packages, because often you will want to run your upload service as a separate application. There are several options supported by blueimp, such as [Java, ASP, Ruby and more](https://github.com/blueimp/jQuery-File-Upload/wiki) or even [node.js Express server](https://www.npmjs.org/package/blueimp-file-upload-node) installed via NPM. If you wish to use self standing server, install only the *tomi:upload-jquery* package.

# Configuration
### Server
First, we need to initialise the server and configure upload paths:

```javascript
//file:/server/init.js
Meteor.startup(function () {
  UploadServer.init({
    tmpDir: '/Users/tomi/Documents/Uploads/tmp',
    uploadDir: '/Users/tomi/Documents/Uploads/',
    getDirectory: function(file, formData) {
      return formData.contentType;
    },
    finished: function(file, folder, formFields) {
      console.log('Write to database: ' + folder + '/' + file);
    }
  })
});
```

Following *options* are available for *UploadServer.init(options)*:

| Field        | Type | Default  | Description  |
| ------------- | ------------- | ----- | ------- |
| tmpDir | String | null | Temporary upload directory
| uploadDir | String | null | Path to the upload directory  
| uploadUrl | String | '/upload/' | Upload route
| maxPostSize | int | 11000000000 | Maximum post size (11 GB)
| minFileSize | int | 1 | Minimum file size
| maxFileSize | int | 10000000000 | Maximum file size (10 GB) 
| acceptFileTypes | RegEx | /.+/i, | Accepted types of files (e.g. prohibit .exe)
| imageTypes | RegEx | /\.(gif\|jpe?g\|png)$/i | Images which can be resized with *Imagemagick*
| imageVersions | Object | {} | Defines the sizes of images which will be converted and saved to upload directory. For example `{thumbnailBig: {width: 400, height: 300}, thumbnailSmall: {width: 200, height: 100}}` | 
| getDirectory | function |  | functions which decides the subdirectory in which the file will be saved. In this function is not defined, no sub-directory is created. For example: `function(file, formData) { return '/my/sub/directory';` }
|getFileName | function |  | Renames the file on the server. In no function is specified, file is saved with the original file name. For example: `function(file, formData) { return 'Saved-' + file; }
| finished | function | | Callback 

### Client

On client we have a possibility to use pre-defined templates, existing for semantic-ui and bootstrap. We can use several upload forms on the page. We can distinguish what kind of content we are uploading, by providing the `contentType` parameter. We can also limit the file type available for the file picker using parameter `fileTypes`. Following is an example of how you can include the upload form on your page:

```html
<template name="home">
    <h3>PDFs</h3>
    {{> upload_semantic_ui contentType='pdfs' fileTypes='.pdf' }}
    <h3>Images</h3>
    {{> upload_bootstrap contentType='images' fileTypes='.jpg' }}
</template>
```

If you wish to create your own template for uploading, you can use following call in the `rendered` callback to set up uploading functionality:

```javascript
Template['your_upload_template'].rendered = function () {
  Uploader.render.call(this);
};
```

When using this approach, your template MUST contain following:

* element with class `.uploadFilePicker` holding the file picker
* element with class `.uploadProgressHolder` holding the file picker
* element with class `.uploadProgressBar` showing the progress bar
* element with class `uploadProgressLabel` showing the progress label

If you wish to use custom URL for your uploads this can be configured as following:

```javascript
Uploader.uploadUrl = 'http://yoururl';
```




