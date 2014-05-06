# MediaElement.js Plugin for Preview Thumbnails

Hover over the time rail on a [MediaElement.js](http://mediaelementjs.com/) player and see video thumbnails.

[See a video of this in action](http://jronallo.github.io/mep-feature-time-rail-thumbnails/)

## Use

To use this feature you will first need to create an image sprite and a metadata WebVTT file.

### Create an Image Sprite

The idea is to take a snapshot of your video every 5 seconds and then stitch the images together into a sprite. This means that only one image file needs to be requested from the server and switching between thumbnails can be very quick. 

Here's one way you could create the image sprite using ffmpeg and montage (from ImageMagick):

```
ffmpeg -i "video-name.mp4" -f image2 -vf fps=fps=1/5 video-name-%05d.jpg
montage video-name*jpg -tile 5x -geometry 150x video-name-sprite.jpg
```

First ffmpeg takes a snapshot of your video every 5 seconds. Then montage reduces each image to 150px across and tiles them from left to right 5 across.

### Create a WebVTT metadata file

Once you have the image sprite you'll also need to create a [WebVTT](http://docs.webplatform.org/wiki/concepts/VTT_Captioning) metadata file that contains the URL to your image sprite. For each time range of 5 seconds the URL is given including a [spatial Media Fragment](http://www.w3.org/TR/media-frags/) hash. This gives information about where in your sprite to look for the thumbnail for that time range. Here's an example of what one looks like:

```
WEBVTT

00:00:00.000 --> 00:00:05.000
http://example.com/video-name-sprite.jpg#xywh=0,0,150,100

00:00:05.000 --> 00:00:10.000
http://example.com/video-name-sprite.jpg#xywh=150,0,150,100

00:00:10.000 --> 00:00:15.000
http://example.com/video-name-sprite.jpg#xywh=300,0,150,100

00:00:15.000 --> 00:00:20.000
http://example.com/video-name-sprite.jpg#xywh=450,0,150,100

00:00:20.000 --> 00:00:25.000
http://example.com/video-name-sprite.jpg#xywh=600,0,150,100

00:00:25.000 --> 00:00:30.000
http://example.com/video-name-sprite.jpg#xywh=0,100,150,100
```

#### Current Sprite Limitations
- All the images embedded in the sprite should have exactly the same dimensions. The styling for the thumbnail hover area is calculated based on the dimensions of the first URL in the WebVTT file.
- Images should only be taken every 5 seconds. While this ought to be configurable in the future it currently is not.

### Markup

Add a new track element to your video element like the following:

```html
<track kind="metadata" class="time-rail-thumbnails" src="http://example.com/video-name-sprite.vtt"></track>
```

The kind attribute must be "metadata" and the class must be "time-rail-thumbnails". The WebVTT file should also be accessible via an AJAX request, so either have it on the same domain or allow cross-origin requests.

### Initialization

Add 'timerailsthumbnails' as the last feature when initializing the mediaelement player:

```javascript
$('video').mediaelementplayer({
   features: ['playpause','progress','current','duration','tracks','volume', 'timerailthumbnails']
});
```

## Installation

This JavaScript plugin is made available as both a Rails Engine gem for the asset pipeline (unreleased) and a bower package. Choose your poison.

### vtt.js

This plugin relies on the [vtt.js](https://github.com/mozilla/vtt.js/tree/master) library for WebVTT parsing. You'll need to include this before including mep-feature-time-rail-thumbnails.js.

### Rails

Include it in your Gemfile:
```ruby
gem 'mep_feature_time_rail_thumbnails'
```

Add it to your application.js:
```javascript
//= require mep-feature-time-rail-thumbnails
```

### Bower

Install with bower:
```
bower i mep_feature_time_rail_thumbnails
```

The file you want is at `bower_components/mep_feature_time_rail_thumbnails/vendor/assets/javascripts/mep-feature-time-rail-thumbnails.js`

The latest vtt.js is not currently available from bower.

## TODO

- Make the interval of thumbnails configurable or really use the timestamps in the WebVTT file rather than relying on 5 second increments.
- Allow to work over HTTP and HTTPS.
- Update bower.json dependencies and instructions when [this bug](https://github.com/mozilla/vtt.js/issues/311) is closed.

## Author

Jason Ronallo

## License

This project rocks and uses MIT-LICENSE.