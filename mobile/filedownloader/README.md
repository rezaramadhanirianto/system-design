# File Downloader

## <a href="technical.md">Technical Documentation</a>

## Gathering Requirements
- Are we designing a part of an application or a general-purpose library?
- Are we downloading a file from the internet and saving it to the disk?
- Should we support pausing/resume/canceling/listing downloads?
- Do we need to support simultaneous file downloads like youtube playlist?
- Do we need to handle authentication?

<image src="assets/design.png" width="500"/>

## Info
- Download Task represent url to download with some listener as Download Listener using builder pattern
- Download library represent main component which contains all component
- Download Queue represent queue of list download with status like pending loading finished etc.
- Download Manager represent component that reponsible to download from http save to disk and save info to local database.

