# Open Source Encore: WebSaver

Recently, I have been playing around with a spare MBP that I have lying around,
trying to find some use for it.

I have a very slick Orange Pi external screen which snugly fits into the Ikea
Kallax line shelf (with a right angle USB C adapter) and I decided to shelve the
laptop and the screen into one of the Kallax compartments and have it serve as
an always-on dashboard.

One downside of the Orange Pi screen is that it needs to use HDMI (which is
impossible to find a small enough right angle adapter for to still fit into the
shelf compartment) or if using USB C, two USB Cs need to be used, one for power
and one for video.

When only one USB C is used, the screen is on half brightness.
This is okay for a dashboard-type screen but it also won't let the screen go
over 720p, that's only allowed when two USB Cs are used at which point the
screen can go to its full 1080p.

While testing this setup out, I wanted to throw some content onto the screen as
I work on the dashboard to display.
At the same time, I was thinking about how to make this whole thing secure.
Ideally, the MBP would be displaying a local web page but otherwise be unusable.
Something like an iPad kiosk mode:

<https://support.apple.com/en-us/HT202612>

Getting a Macbook to run in kiosk mode is much more involved and I didn't like
the approaches I found online.
But there was one thought that I found enticing:

Use a screensaver which would be displaying a webpage.

I looked around and sure enough, there is one such screensaver and it has a
pretty old legacy as well, it seems!

<https://github.com/brockgr/websaver>

When I found this repository, there was only a 2015 release and instructions on
how to build it and use it were sparse, so I set out to figure this out and then
I opened a PR with improved instructions here:

<https://github.com/brockgr/websaver/pull/26>

This helped me to build and install the wallpaper, but I still found myself
wishing I didn't even need to use XCode and could just grab the software.
This motivated me to try and contribute a GitHub Actions workflow which would
build the screensaver and upload the built package as an artifact of the
workflow as well as cut a new repository release for people to be able to grab
the app from.

And that's what I did:

<https://github.com/brockgr/websaver/pull/27>

This PR took a while to crack and in the process I learned a few things:

- The build directory is in `/Users/$user/Library/Developer/Xcode/DerivedData`

- The directory itself is named after the project/schema and some GUID

- The GUID appears to be stable across build with no changes but I don't know
  if it is stable across changes to the codebase XCode sees (ObjectiveC/Swift)

- It is not trivial to make `zip` bundle a directory which is not in the current
  working directory such that the paths are relative in a nice way

- GitHub Releases need to be tied to a tag and there is no maintained official
  GitHub Action for cutting releases and the 3rd party ones do not seem to allow
  you to have them create a one-off tag without you needing to worry about it
  
  I ended up adding a few lines of Bash to create a tag based on the current
  date and time and tagging the repository before creating the Release that
  would be associated to the tag.
  
  The <https://github.com/rickstaa/action-create-tag> Action could not be used
  because it doesn't run on macOS.

I also learnt some WebSaver-specific things.
The app/screen saver is built for Intel processors and it is not a Universal
binary (which is hardly shocking considering how old the code base is) but I
think it should be possible to extend it to work on Apple processors later on.

Also, downloading screen savers is no different from downloading apps and I have
not found a nicer way to make the screen saver runnable than to run this CLI on
the downloaded package before opening it:

`xattr -d com.apple.quarantine WebSaver.saver`

I don't think it is possible to do this in the workflow because the quarantine
attribute is probably applied to the file upon it being downloaded or extracted
from a downloaded archive.

I am not sure my PR will be accepted upstream.
The repository might not be maintained anymore.
But there was a relatively recent PR which boosted the support to macOS 10.13
and that one got merged, so there is some hope.

There is a few things I want to get into in case my PRs do get merged:

- Update the README to mention you can grab the screen saver in GitHub Releases
  but you need to run the `xattr -d com.apple.quarantine` command
  
  There is the trick with going to System Settings > Privacy & Security and
  scrolling down to the warning about an app that was prevented from running
  where you can choose Open Anyway and that clears the quarantine bit, but I
  have not found it to work on screensavers for some reason.
  
  Also I am aware of a trick where you can right-click the downloaded package
  and select Open (instead of double-clicking it) and that should clear the
  quarantine bit as well, but again, in this case I have not found that to work.

- Update the XCode project to built a Universal binary or maybe add a new schema
  (or whatever mechanism there is in XCode for build configuration) and have two
  targets - Intel and Apple - and built both in the CI
  
  The screen saver is tiny either way so the Universal binary would probably be
  just fine.
  
  <https://developer.apple.com/documentation/apple-silicon/building-a-universal-macos-binary>
  
- Add support for displaying local `file:` protocol files.
 
  This is important to be and it doesn't work out of the box.
  I've already tried a few things to make it work but to no avail.
  See <https://github.com/brockgr/websaver/issues/25>
  
- Notarize the application to get rid of the security warning

  <https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution>
  
  This will probably always remain a pipe dream as I have no experience with
  this but it would be very cool to achieve!
  
I am hoping my contributions to the project do get accepted and I make it nicer
to download and get the screen saver to run for everyone.

Separately, I am hoping I can get this screen saver to show local files, because
unlike the above, this is actually a blocker for my dashboard idea.
I don't want to leave my Mac running 24/7 unattended with my user account logged
in so some sort of a security solution is needed and a wallpaper like this would
work just perfectly.
