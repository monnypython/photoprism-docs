# Frequently Asked Questions

### What are sidecar files and where do I find them? ###

A sidecar is a file which sits **alongside** your main photo or video files, 
typically using the same name, and a different extension like 

 * `IMG_0101.jpg`
 * `IMG_0101.json`
 * `IMG_0101.yaml`

New sidecar files will be created in the *storage* folder by default so that the *originals* folder 
can be mounted read-only.

!!! info
    PhotoPrism will always look out for existing sidecar files and use them for indexing, 
    even if `PHOTOPRISM_DISABLE_EXIFTOOL` and `PHOTOPRISM_DISABLE_BACKUPS` are set to `"true"`.

Three types of metadata sidecar files are supported currently:

#### JSON ####

If not disabled via `PHOTOPRISM_DISABLE_EXIFTOOL` or `--disable-exiftool`, [Exiftool](https://exiftool.org/) is used to 
automatically create a JSON sidecar for each media file. 
**This way, embedded XMP and video metadata can be indexed as well.**
Native metadata extraction is limited to common Exif headers.
Note that this causes moderate overhead when indexing for the first time.

JSON files may also be useful for debugging as they contain the complete metadata, 
and can be processed using common development tools and text editors.

!!! tip
    PhotoPrism can also read JSON files exported by Google Photos. Support for additional
    schemas may be added over time.

#### YAML ####

If not disabled via `PHOTOPRISM_DISABLE_BACKUPS` or `--disable-backups`, PhotoPrism will automatically create / update 
YAML sidecar files while indexing and after manually editing fields like title, date, or location. 
They **serve as a backup** in case the database (index) gets lost, or when folders are synced with a remote 
PhotoPrism instance.

Like JSON, YAML files can be opened using common development tools and text editors.
Changes won't be synced back to the original index though as this might overwrite existing data.

#### XMP ####

XMP (Extensible Metadata Platform) is an XML-based metadata container format 
[invented by Adobe](https://www.adobe.com/products/xmp.html). 
It offers much more fields (as part of embedded models like Dublin Core) than Exif. 
That also makes it difficult - if not impossible - to provide complete support.
Reading Title, Copyright, Artist, and Description from XMP sidecar files is implemented as a proof-of-concept, 
[contributions welcome](../developer-guide/metadata/xmp.md).
Indexing embedded XMP is only possible via Exiftool, see above.

### Which folder will be indexed? ###

This depends on your [runtime environment](docker-compose.md) and [configuration](config-options.md).
While sub-folders can be selected for indexing in the UI, changing the *originals* base folder 
requires a restart for security reasons.

If you skip configuration and don't use one of our Docker images, PhotoPrism will try to find 
a photo library by going through a list of common 
[folder names](https://github.com/photoprism/photoprism/blob/develop/pkg/fs/dirs.go) 
like `/photoprism/originals`, `Pictures`, and `~/Photos`. It will also search for other resources
like external applications, classification models, and frontend assets.

Your library will be mounted from `~/Pictures` by default when using our 
example [docker-compose.yml](docker-compose.md) file, where `~` is a placeholder for your home directory. 

You may mount any folder accessible from your computer, including network drives.
Note that PhotoPrism won't be able to see folders that have not been mounted unless you install it locally
without Docker (developers only).

Multiple folders can be indexed by mounting them as sub-folders of `/photoprism/originals`:

```
volumes:
  - "~/Family:/photoprism/originals/Family"
  - "~/Friends:/photoprism/originals/Friends"
``` 

### Which file types are supported? ###

PhotoPrism's primary image file format is JPEG.
While indexing, a JPEG sidecar file may automatically be created for RAW, HEIF, TIFF, PNG, BMP, 
and GIF files. It is required for classification and resampling.

Support for specific RAW formats depends on the runtime environment and configuration. PhotoPrism may use 
[Darktable](https://www.darktable.org/) and [RawTherapee](https://rawtherapee.com/) for RAW to JPEG conversion. 
On Mac OS, [Sips](https://ss64.com/osx/sips.html) can be used as well.

We support [all common video types](../developer-guide/media/videos.md).
You should configure PhotoPrism to automatically create JSON sidecar files so that
video metadata like location and duration can be indexed.

You're welcome to open an issue if you experience issues with a specific file format.

### Why don't you display animated GIFs natively? ###

PhotoPrism focuses on photographic images and short videos. You may
[convert your GIF files to  H.264 / MPEG-4 AVC](https://unix.stackexchange.com/questions/40638/how-to-do-i-convert-an-animated-gif-to-an-mp4-or-mv4-on-the-command-line) 
using `ffmpeg`. That's also what Twitter does when you post a GIF. They will then be shown as 
"live photos" and start playing on mouse over while also consuming less storage and bandwidth 
compared to your original GIF files.

### Why is my storage folder so large? What is in it? ###

The storage folder contains sidecar, thumbnail, and configuration files.
It may also contain index database files if you're using SQLite.
Most space is consumed by thumbnails: These are high-quality resampled, smaller 
versions of your originals.

Thumbnails are required because Web browsers do a pretty bad job at resampling large images 
so that they fit your screen. Using originals for slideshows and search result previews 
would consume much more browser memory, and reduce overall performance, as well.

If you're happy with lower quality thumbnails, you can reduce their JPEG quality 
and/or set a size limit. Note that existing thumbnail files won't be replaced automatically 
after changing [config values](config-options.md).

You may also choose to render thumbnails on-demand if you have a fast CPU and enough memory. 
However, storage is typically affordable enough for most users to go for better quality and 
performance instead.

### Can I skip creating thumbnails completely? ###

The smallest [configurable](../user-guide/settings/advanced.md) limit is 720px for consumption 
by the indexer and TensorFlow during image classification. Recreating and keeping them in memory
is too demanding, even for the most powerful servers.

### I'm having issues understanding the difference between the import and originals folders? ###

Import is a temporary folder from which you can move or copy files to *originals* in a structured way that avoids duplicates.
Most users with existing collections will want to index their *originals* folder without import, 
so that existing file and directory names don't change. On the other hand, importing may be more efficient when
adding files as you don't need to re-index *originals*.

### What exactly does the read-only mode? ###

There are users who don't want us to modify their original files and folders in any way, so we've added
a configuration option for this use case. It will disable uploads, import and future features
that might rename, update or delete files in the *originals* folder.

### I could not find a documentation of config parameters? ###

You may run `photoprism help` in a terminal to see all options and commands. 
We also maintain a complete list of [config options](config-options.md) in these docs.

Our Docker Compose [examples](https://dl.photoprism.org/docker/docker-compose.yml) are continuously maintained and inline documentation 
has been added to simplify installation.

### Can I install PhotoPrism in a sub-directory on a shared domain?

This is possible with our latest release if you run it behind a proxy.
Note that for progressive web apps to work as designed, the service worker should 
be located in the root directory. Also keep in mind sharing a domain with
other apps may negatively impact the performance and 
[security](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
of all apps installed.  Sharing links will be longer as well.

### Why is PhotoPrism getting stuck in a restart loop? ###

These restarts are triggered by Docker (based on your configuration)
when PhotoPrism is unable to start.
They are typically caused by one or more of the following reasons:

1. Your (virtual) server disk may be full.
2. The storage folder may not be writable.
3. Your database server may be unavailable.
4. There are connection issues caused by a proxy or firewall.
5. Kernel security modules such as
   [AppArmor](https://wiki.ubuntu.com/AppArmor) or
   [SELinux](https://en.wikipedia.org/wiki/Security-Enhanced_Linux)
   may be blocking permissions.

Please check the server logs for a detailed error message like "disk full" or "wrong permissions".
If you're using Docker Compose, you may enter this command to see the last 20 log entries:

```
docker-compose logs --tail=20
```

Linux kernel security may be disabled on private servers, especially if you have no experience
with configuring it properly. Use the commands `chmod` and `chown` to fix file system permissions
on Linux and macOS.
Available disk space can be displayed with `df -h`. The size of virtual disks and memory can be
increased in Docker settings.

### How can I uninstall PhotoPrism? ###

This depends on how you installed it. If you're running PhotoPrism with Docker Compose, 
this command will stop and remove the Docker container:

```
docker-compose rm -s -v
```

Please refer to the official Docker [documentation](https://docs.docker.com/compose/reference/rm/) 
for further details.

### How can I mount network shares with Docker? ###

There are multiple ways of using network storage. 
One of the easiest might be to directly mount NFS shares with Docker.

You can mount any number of NFS shares as folders. Follow this `docker-compose.yml` example 
if you want to mount the *originals* folder as a share:

```yaml
services:
  photoprism:
    # ...
    volumes:
      # Map originals folder to NFS:
      - "photoprism-originals:/photoprism/originals"     

volumes:
  photoprism-originals:
    driver: local
    driver_opts:
      type: nfs
      # The IP of your NAS:
      o: "username=user,password=secret,addr=1.2.3.4,soft,rw"
      # Share path on your NAS:
      device: ":/mnt/photos" 
```

For Windows / CIFS shares:

```yaml
volumes:
  photoprism-originals:
    driver: local
    driver_opts:
      type: cifs
      o: "username=user,password=secret,rw"
      device: "//host/folder"
```

!!! info 
    This was tested with TrueNAS and NFS, but other (network) file systems may be mounted with Docker as well.

!!! tip 
    Mounting the *import* folder to a share which is also accessible via other ways (e.g. CIFS)
    is especially handy, as you can dump all data from a SD card / camera directly into that folder 
    and trigger the index in the GUI afterwards. So you can skip the upload dialog in the 
    GUI and it's a little faster.

### I'm using an operating system without Docker support. How to install and use PhotoPrism without Docker? ###

In general, you would build / install it like a [developer](../developer-guide/setup.md) since we don't have packages 
for specific operating systems yet.

Instead of using Docker, you can manually type the commands listed in our development 
[Dockerfile](https://github.com/photoprism/photoprism/blob/develop/docker/development/Dockerfile) and replace packages with 
what is available in your environment. You often don't need to use the exact same versions for dependencies.

If your operating system has Docker support, we recommend learning Docker as it vastly simplifies installing
and upgrading.

### Do you support Podman? ###

Podman works just fine both in rootless and under root. Mind the SELinux which is enabled on 
Red Hat compatible systems, you may hit permission error problems. 

More details on on how to run PhotoPrism with [Podman](https://podman.io/) on CentOS in 
[this blog post](https://lukas.zapletalovi.com/2020/01/deploy-photoprism-in-centos-80.html), 
it includes all the details including root and rootless modes, user mapping and SELinux.

### Do you provide LXC images? ###

There is currently no [LXC](https://linuxcontainers.org/) build for
PhotoPrism, see [issue #147](https://github.com/photoprism/photoprism/issues/147) for details.

### Any plans to add support for Active Directory, LDAP or other centralized account management options? ###

There is no single sign-on support yet as we didn't consider it essential for our initial release.
Our team is currently working on [OpenID Connect](https://github.com/photoprism/photoprism/issues/782),
which will be available in a future release.

!!! info
    Our development and testing efforts are focused on small servers and home users. Adding functionality
    that is primarily useful for business environments, or that only benefits few private 
    users with special needs, diverts resources away from features that benefit everyone.
    Professional users are welcome to [reach out](../contact.md) to us for a custom solution.
