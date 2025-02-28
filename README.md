Telepathy: An OSINT toolkit for investigating Telegram chats. Developed by Jordan Wildon. Version 2.3.2.

Telepathy has been described as the "swiss army knife of Telegram tools," allowing OSINT analysts, researchers and digital investigators to archive Telegram chats (including replies, media content, comments and reactions), gather memberlists, lookup users by given location, analyze top posters in a chat, map forwarded messages, and more.

The toolkit has already seen a wide variety of use cases, including but not limited to: in investigative and data journalism, by academic and research institutions, and for intelligence gathering and analysis.


## !! IMPORTANT: 
With the update to 2.3.0, you will need to delete your login.txt file to prevent errors if using the alternative login feature. Upon first use, Telepathy will guide you through setup of the details once again. To work around this, instead of deleting and recreating the file, you can add a newline character to the end of your current API details to ensure Telepathy scans the file correctly.

A note on unique identifiers per account: You will notice that depending on which alternative account you use, the access hash will vary. The same will happen with User IDs, which are unique to each Telegram account accessing them. For deeper data analysis based on user IDs, this is important to bare in mind as users will have as many unique IDs as accounts you've used to access information. In future, Telepathy may include a feature to assign unique identifier per account found based on a hash of the available information, regardless of which account accessed the data.



## Installation

### Pip install (recommended)

```
$ pip3 install telepathy
```

### Install from source

```
$ git clone https://github.com/jordanwildon/Telepathy.git
$ cd Telepathy
$ pip install -r requirements.txt
```

## Setup

On first use, Telepathy will ask for your Telegram API details (obtained from my.telegram.org). Once those are set up, it will prompt you to enter your phone number again and then send an authorization code to your Telegram account. If you have two-factor authentication enabled, you'll be asked to input your Telegram password.

OPTIONAL: Installing cryptg ($ pip3 install cryptg) may improve Telepathy's speed. The package hand decryption by Python over to C, making media downloads in particular quicker and more efficient. 


## Usage:

```
telepathy [OPTIONS]
```

Options:
- **'--target', '-t' [CHAT]**

this option will identify the target of the scan. The specified chat must be public or have a private link. To get the chat name, look for the 't.me/chatname' link, and subtract the 't.me/'.

For example:

```
$ telepathy -t durov
```

The default is a basic scan which will find the title, description, number of participants, username, URL, chat type, chat ID, access hash, first post date and any applicable restrictions to the chat. For group chats, Telepathy will also generate a memberlist (up to 5,000 members).


- **'--comprehensive', '-c'**

A comprehensive scan will offer the same information as the basic scan, but will also archive a chat's message history, gather the number of reactions, archive how many times a message has been forwarded, the number of replies to each message, and more.

Reaction lists are included in the archive file, including basic calculations of engagement rate. Only the most-common reactions are listed, with the total including all possible reactions. Currently, Telepathy calculates engagement rates based on forwards, comments and reactions seperately, with a calculation based on post views and one based on chat participant count. In future, Telepathy may include deeper analytics which can be cross-compared between chats based on a combination of these metrics, fixing for when comments, reactions or forwards are allowed or disallowed in a given chat.

For example:

```
$ telepathy -t durov -c
```


- **'--forwards', '-f'**

This flag will create an edgelist based on messages forwarded into a chat. It can be used alongside either a default or comprehensive scan. Since 2.3.0, Telepathy now formats these edgelists to maximize compatability with Gephi.

For example:

```
$ telepathy -t durov -f

$ telepathy -t durov -c -f
```


- **'--media', '-m'**

Use this flag to include media archiving alongside a comprehensive scan. This makes the process take significantly longer and should also be used with caution: you'll download all media content from the target chat, and it's up to you to not store illegal files on your system.

To archive media, you must run a comprehensive scan:

```
$ telepathy -t durov -c -m
```

Once files have downloaded, you can run exiftool on the associated media directory to gather deeper insights on the files, their metadata, and in some cases attribute who might be behind an anonymous channel. Further details are in the "bonus investigations tips" section of this README.


- **'--user', '-u'**

Looks up a specified user. This will only work if your account has "encountered" the user before (for example, after archiving a group), you can specify User ID or @nickname. If looking up by username, it's not always necessary for your account to have already seen the user.

```
$ telepathy -t 0123456789 -u

$ telepathy -t @test_user -u
```


- **'--location', '-l']**

Finds users near to specified coordinates. Input should be longitude followed by latitude, seperated by a comma. This feature only works if your Telegram account has a profile image which is set to be publicly viewable.

While searches for multiple locations at once may work in some cases, Telegram appears to have a limit on how quickly an account can cycle through locations. At the time of writing, this appears to be at least ten minutes. Further location scanning support while using multiple accounts is being explored for a future release.

```
$ telepathy -t 51.5032973,-0.1217424 -l
```


- **'--alt', '-a' [NUMBER]**

Flag for running Telepathy from an alternative number or API details. You can use the same API key and Hash but authenticate with a different phone number. This allows for running multiple scans at the same time. Telepathy will default to the first details you offer, and up to four others can be added. Please see the notes at the top of this README for information regarding limitations with user IDs using this method.

```
$ telepathy -t Durov -c -a 1
```


- **'--export', '-e'**

Exports all chats your account is part of to a CSV file. In a future release, this may assist with provisioning new accounts to automatically following the listed groups.

```
$ telepathy -e
```
  

- **'--reply', '-r'**

Flag for enabling channel reply retrieval, this will archive replies and list users who replied to messages in the target channel. 

```
$ telepathy -t [CHANNEL] -c -r 
```


- **'--translate', '-tr'**

Flag for enabling auotmatic translation (currently only into English) during message retrieval.

```
$ telepathy -t [CHANNEL] -c -tr
```


## Bonus investigations tips:

 - Navigating to a media archive directory and running Exiftool may give you a whole host of useful information for further investigation. Telegram doesn't currently scrub metadata from PDF, DOCX, XLSX, MP4, MOV and some other filetypes, which offer creation and edit time metadata, often timezones, sometimes authors, and general technical information about the perosn or people who created a media file.  
 ```
$ cd ./telepathy/telepathy_files/CHATNAME/media
$ exiftool * > metadata.txt
```
 - Group and inferred channel memberlists offer a point of further investigation for usernames found. By using [Maigret](https://github.com/soxoj/maigret), you can look up where else a username has been used online. While this is not accurate in all cases, it's been proven to be helpful for identifying where a person has reused handles across platforms. In this case, remember to verify your findings to avoid false positives.


## A note on how Telegram works

Telegram chats are organised into three key types: Channels, Megagroups/Supergroups and Gigagroups. Each option works slightly differently depending on the chat type. Channels can have seemingly unlimited subscribers and are where an admin will broadcast messages to an audience, Megagroups can have up to 200,000 members, each of whom can participate (if not restricted), and Gigagroups sit somewhere between the two.


## Upcoming changes
In some environments (particularly Windows), Telepathy struggles to effectively manage files and can sometimes produce errors. Fixes for these errors will come in due course.

Upcoming features include:

  - [ ] Adding a time specification flag to set archiving for specific period.
  - [x] The ability to gather the number of reactions to messages, including statistics on engagement rate.
  - [ ] Finding a method to once again gather complete memberlists (currently restricted by the API).
  - [ ] Improved statistics: including timestamp analysis for channels.
  - [ ] Generating an entirely automated complete report, including visualisation for some statistics.
  - [ ] Hate speech analytics.
  - [ ] Include sockpuppet account provisioning (creation of accounts from previous exported lists).
  - [ ] Listing who has group admin rights in memberlists.
  - [ ] Media downloaded in the background to increase efficiency or progress bars for media downloads to give a better estimation of runtime.
  - [x] When media archiving is flagged, the location of downloaded content will be added to the archive file.
  - [ ] Exploring, and potentially integrating, media cross-checks based on https://github.com/conflict-investigations/media-search-engine.
  - [ ] Ensuring inferred channel memberlists don't contain duplicate entries.
  - [ ] Introducing local chat retrival within the location lookup module.
  - [x] Further code refactoring to ensure long-term maintainability.
  - [x] Adding additional alternative logins.
  - [ ] Improved language support.
  - [ ] Correctly define destinction between reply (as in a chat) and comment (as in channel).
  - [ ] Exploration of whether channel events can be included, such as name changes.
  - [x] Including last seen on user lookup.

## feedback

Please send feedback to @jordanwildon on Twitter. You can follow Telepathy updates at @TelepathyDB.


## Usage terms

You may use Telepathy however you like, but your usecase is your responsibility. Be safe and respectful.


## Credits

All tools created by Jordan Wildon (@jordanwildon). Special thanks go to [Giacomo Giallombardo](https://github.com/aaarghhh) for adding additional features and code refactoring, [jkctech](https://github.com/jkctech/Telegram-Trilateration) for collaboration on location lookup via the 'People Near Me' feature, and Alex Newhouse (@AlexBNewhouse) for his help with Telepathy v1. Shoutout also to [Francesco Poldi](https://github.com/pielco11) for being a sounding board and offering help and advice when it comes to bug fixes.

Where possible, credit for the use of this tool in published research is desired, but not required. This can either come in the form of crediting the author, or crediting Telepathy itself (preferably with a link).
