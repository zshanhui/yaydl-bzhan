# yaydl (forking to add support for B站)

CURRENT TODOs:

  - add support for Bilibili
  - remove unused sites
  - add Playwright for robustness

[![Crates.io](https://img.shields.io/crates/v/yaydl)](https://crates.io/crates/yaydl)

**y**et **a**nother **y**outube *(and more)* **d**own **l**oader

    % yaydl "https://www.youtube.com/watch?v=jNQXAC9IVRw"

## How? What? Why?

    % yaydl --help

# Features

* Can download videos.
* Can optionally keep only the audio part of them.
* Could convert the resulting file to something else (requires the `ffmpeg` binary).
* Comes as a single binary (once compiled) - take it everywhere on your thumbdrive, no Python cruft required.

## Currently supported sites

* invidious (most of them) · porndoe.com · pr0gramm.com · spankbang.com · Vidoza · Vimeo · vivo.sx · voe.sx · watchdirty.to · xhamster.com · YouTube
* Also, direct downloads for `.mp4` and `.mpg` files.

There is an easy way to add more supported sites, see below for details.

## Non-features

The list of features is deliberately kept short:

* No output quality choice. `yaydl` assumes that you have a large hard drive and your internet connection is good enough, or else you would stream, not download.
* No complex filters. This is a downloading tool.
* No image file support. Videos only.

## How to install

### Prebuilt Windows binaries

Available on [cdn.tuxproject.de](https://cdn.tuxproject.de/projects/yaydl/).

### From the source code

Install Rust (e.g. with [rustup](https://rustup.rs)), then:

**using Fossil:**

    % fossil clone https://code.rosaelefanten.org/yaydl
    % cd yaydl
    % cargo build --release

**using Git:**

    % git clone https://github.com/dertuxmalwieder/yaydl
    % cd yaydl
    % cargo build --release

### From Cargo

    % cargo install yaydl

### From your package manager

`pkgsrc` (with `pkg_add`):

    % pkg_add yaydl

`pkgsrc` (with `pkgin`):

    % pkgin install yaydl

`xbps-install` on Void Linux:

    % xbps-install yaydl

Other package managers:

* Nobody has provided any other packages for `yaydl` yet. You can help!

# How to use the web driver (very beta, at your own risk!)

For some video sites, `yaydl` needs to be able to parse a JavaScript on them. For this, it needs to be able to spawn a headless web browser. It requires Google Chrome, Microsoft Edge or Mozilla Firefox to be installed and running on your system.

1. Install and run [ChromeDriver](https://chromedriver.chromium.org) (if you use Chrome), the *typically* named [Microsoft Edge WebDriver](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/) (if you use Edge) or [geckodriver](https://github.com/mozilla/geckodriver/releases) (if you use Firefox) for your platform.
2. Tell `yaydl` that you have a web driver running: `yaydl --webdriver <port> <video URL>`. (The drivers usually run on port 4444 or 9515, please consult their documentation if you are not sure.)
   *Hint:* If you need this feature regularly, you can also use the environment variable `YAYDL_WEBDRIVER_PORT` to set the port number for all further requests.
3. In theory, it should be possible to use more sites with `yaydl` now. :-)

# How to set the default Invidious instance

For rather obvious reasons, including (but not limited to) the fact that Google tries to choke third-party clients, `yaydl` has been using Invidious as a wrapper since version 0.13.0. Now sometimes, the default instance hard-coded into `src/handlers/youtube.rs` *will* fail to work properly. You can use the environment variable `YAYDL_INVIDIOUS_INSTANCE` to change that: Just set it to the URI (including "https://") of [any other instance](https://docs.invidious.io/instances/).

# How to contribute code

1. Read and agree to the [Code of ~~Conduct~~ Merit](CODE_OF_CONDUCT.md).
2. Implicitly agree to the [LICENSE](LICENSE). Nobody reads those. I don't either.
3. Find out if anyone has filed a GitHub Issue or even sent a Pull Request yet. Act accordingly.
4. Send me a patch, either via e-mail (`yaydl at tuxproject dot de`), on the IRC or as a GitHub Pull Request. Note that GitHub only provides a mirror, so you'd double my work if you choose the latter. :-)

If you do that well (and regularly) enough, I'll probably grant you commit access to the upstream Fossil repository.

### Add support for new sites

1. Implement `definitions::SiteDefinition` as `handlers/<YourSite>.rs`.
2. Push the new handler to the inventory: `inventory::submit! {  &YourSiteHandler as &dyn SiteDefinition }`
3. Add the new module to `handlers.rs`.
4. Optionally, add new requirements to `Cargo.toml`.
5. Send me a patch, preferably with an example. (I cannot know all sites.)

#### Minimal example that does nothing

```rust
// handlers/noop.rs

use anyhow::Result;

use crate::definitions::SiteDefinition;
use crate::VIDEO;

struct NoopExampleHandler;
impl SiteDefinition for NoopExampleHandler {
    // Parameters sent to the handler by yaydl:
    // - url:            The video page's URL.
    // - webdriver_port: The port that runs the WebDriver client.
    //                   Defaults to 0 if there is no WebDriver configured.
    // - onlyaudio:      true if only the audio part of the video should be
    //                   kept, else false.
    fn can_handle_url<'a>(&'a self, url: &'a str, webdriver_port: u16) -> bool {
        // Return true here if <url> can be covered by this handler.
        // Note that yaydl will skip all other handlers then.
        true
    }

    fn does_video_exist<'a>(&'a self, video: &'a mut VIDEO, url: &'a str, webdriver_port: u16) -> Result<bool> {
        // Return true here, if the video exists.
        Ok(false)
    }

    fn is_playlist<'a>(&'a self, url: &'a str, webdriver_port: u16) -> Result<bool> {
    	// Return true here, if the download link is a playlist.
    	Ok(false)
    }

    fn find_video_title<'a>(&'a self, video: &'a mut VIDEO, url: &'a str, webdriver_port: u16) -> Result<String> {
        // Return the video title from <url> here.
        Ok("".to_string())
    }

    fn find_video_direct_url<'a>(&'a self, video: &'a mut VIDEO, url: &'a str, webdriver_port: u16, onlyaudio: bool) -> Result<String> {
        // Return the direct download URL of the video (or its audio version) here.
        // Exception: If is_playlist() is true, return the playlist URL here instead.
        Ok("".to_string())
    }

    fn find_video_file_extension<'a>(&'a self, video: &'a mut VIDEO, url: &'a str, webdriver_port: u16, onlyaudio: bool) -> Result<String> {
        // Return the designated file extension of the video (or audio) file here.
        Ok("mp4".to_string())
    }

    fn display_name<'a>(&'a self) -> String {
        // For cosmetics, this is the display name of this handler.
        "NoopExample"
    }

    fn web_driver_required<'a>(&'a self) -> bool {
        // Return true here, if the implementation requires a web driver to be running.
        false
    }
}

// Push the site definition to the list of known handlers:
inventory::submit! {
    &NoopExampleHandler as &dyn SiteDefinition
}
```

### Fix some bugs or add new features

1. Do so.
2. Send me a patch.

## Donations

Writing this software and keeping it available is eating some of the time which most people would spend with their friends. Naturally, I absolutely accept financial compensation.

* PayPal: [GebtmireuerGeld](https://paypal.me/gebtmireuergeld)
* Liberapay: [Cthulhux](https://liberapay.com/Cthulhux/donate)

Thank you.

## Contact

* Twitter: [@tux0r](https://twitter.com/tux0r)
* Mastodon: [@tux0r@layer8.space](https://layer8.space/@tux0r)
* Forum: [DonationCoder.com](https://www.donationcoder.com/forum/index.php?topic=50691.0)
* IRC: `irc.oftc.net/yaydl`
