---
layout: post
title:  "Create your own download manager using Node.js"
author: mustapha
date:   2016-05-04 11:21:25
categories: nodejs
tags: nodejs downloads albums javascript
image: /assets/article_images/2016-05-04-create-your-own-download-manager/e.jpg
---

---
You are about to create a download manager, which queues,
starts and repeats over a list of downloadable content.

The example used is a download manager for albums of images.

## The Rule
Album downloads are queued. They are not simultaneous, and this save us from
server overload.

## The Downloader object
Downloader is a singleton class, instantiated by the view component to listen
for events.

{% highlight javascript %}
import EventEmitter from 'events'
class DownloadEmitter extends EventEmitter {}
{% endhighlight %}


{% highlight javascript %}
export default class Downloader {
  constructor() {
    this.downloadPath = 'PATH'

    if(!fs.existsSync(this.downloadPath)) {
      fs.mkdirSync(this.downloadPath)
    }

    this.downloadEmitter = new DownloadEmitter()
  }

  static getInstance() {
    if(!instance) {
      instance = new Downloader()
    }

    return instance
  }
  ...
}
{% endhighlight %}

In the constructor, we create the folder for downloads, initialize EventEmitter, etc.

But what is EventEmitter? Why do we emit events?
View components are waiting for something to happend. Either it is a user
action or background process result.
Thanks to EventEmitter, Donwloader will process those variables to our views.
Users will be happy to see the UI state changing because they only trust what they see.

### The Queue vs The data
The queue will hold albums users are going to download. Once donwloaded,
an album is removed from the queue.

Data can hold everything. But for simplicity sake, we represent it this way.

{% highlight json %}
{
  albums: [
    {
      id: 1,
      images: [
        { url: 'http://i.imgur.com/aWaZqVs.jpg' },
        ...
      ]
    }
    ...
  ]
}
{% endhighlight %}

{% highlight javascript %}
let queue = {}

getAlbum(index) {
  return queue[Object.keys(queue)[index]]
}

addToQueue(album, cb) {
  const self = this

  // XXX Save each album in a different location?
  let dir = path.join(self.downloadPath, (album.id).toString())
  album.dir = dir

  // Add properties to image donwload progress on a given album
  album.downloadStarted = false
  album.downloadProgress = 0
  album.downloadedImages = []

  queue[album.id] = album

  self.downloadQueuedAction(album.id)

  cb()
}

shiftQueue() {
  let album = this.getAlbum()
  delete queue[album.id]
}
{% endhighlight %}

Callback (cb) is a function parameter to execute when we are done with queueing.
Callback can perform for example, UI update. Say, 'Waiting...'

We will see more about 'downloadQueuedAction' later.

### Download: the main part

Request is an awesome HTTP client for node. Its use is not limited to query an api,
but to download data from the Internet, and much more.

In this section, we query the server for images, write them on disk, and move on.

Let's start with download behaviour.

{% highlight javascript %}
downloadRequestWithRetry(options, retry, started, progressed, finished) {
  let self = this
  var filename = (options.id).toString()
  var writeStream = fs.createWriteStream(path.join(options.dir, filename))

  let hasErrors = false
  request(options.file, {timeout: 15000})
  .on('response', function(data) {
    // Only init download started callback first time
    if(retry === 0) {
      started(parseInt(data.headers[ 'content-length' ]))
    }
  })
  .on('error', function(err) {
    // When an error occurs, re initiate download with retry incremented.
    // This supposes we are online.
    hasErrors = true
     if(retry < 5) {
      retry ++
      self.downloadRequestWithRetry(options, retry, started, progressed, finished)
    }
  })
  .on('data', function(chunk) {
    progressed(chunk.length)
  })
  .on('end', function() {
    // We don't call finished callback if an image is not successfully downloaded.
    if(!hasErrors) {
      finished()
    }
  })
  .pipe(writeStream)
}
{% endhighlight %}

I know it is a long method. But is it not simple enough?
First thing that catch your attention is the name of the method.
'Retry'. Well, if an image download fails due to bad networking, we re execute
the same part of the function.

'error' and 'end' are both events, both called when request origins errors.
'end' event is he result of either successfull/failed download.
That's why we add a boolean to check if the download has been finished successfully.
To retrieve the size of the image, we listens for 'response' event.
Finally, 'data' event is triggered as the download progresses.

We need a starting point for our downloads. It looks this way:

{% highlight javascript %}
downloadNext() {
  let self = this

  // Pick the first album on the queue
  let album = self.getAlbum(0)

  self.downloadEmitter.emit('album-download-started')

  let images = album.images

  // Variables to track download progress, totalSize equals to sum of
  // album images size. Received increments itself with every new chunk progress.
  let totalSize = 0
  let received = 0

  images.forEach(function(image) {
    let options = {
      id: (image.id).toString(),
      url: image.url,
      dir: album.dir
    }

    self.downloadRequestWithRetry(options, 0,
      ((size) => {
        totalSize += size
      }),
      ((data) => {
        received += data
        let progress = Math.floor(received / totalSize * 100)

        self.downloadEmitter.emit('album-download-progress', album.id, progress)
      }),
      (() => {
        album.downloadedImages.push(image.id)

        if(album.downloadedImages.length == images.length) {
          self.downloadEmitter.emit('playlist-download-finish', album.id)
          self.downloadFinishedAction()
        }
      })
    )
  })
}
{% endhighlight %}

We carefully notice the emitter part.
We emit an events when we start downloading an album, to switch from waiting
state to downloading. Then we emit the progress event to print
progression percentage.

Starting point 'downloadNext' is called when first album is queued, or to process
download for pending requests.

Methods 'downloadQueuedAction' and 'downloadFinishedAction' are there to initiate
main download action over the queue object.
You will notice that we also used underscore helpers to check if our queue object is empty.
So don't forget to import it.

{% highlight javascript %}
  donwloadQueuedAction() {
    const self = this
    if(isDownloading) {

    }else {
      self.downloadNext()
      isDownloading = true
    }
  }

  downloadFinishedAction() {
    const self = this
    self.shiftQueue()
    if(_.isEmpty(queue)) {
      isDownloading = false
    } else {
      self.downloadNext()
    }
  }
{% endhighlight %}

## Conclusion
We talked about components and views a bit. We could have used React or Vue.js, two
awesome libraries for constructing views. Maybe we will add the UI part in the future.

Doing that way with downloads, we considerably avoided error responses from the server.
So it is much safer to process download of one album, and queue others.
