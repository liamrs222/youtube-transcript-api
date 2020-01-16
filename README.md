
# YouTube Transcript/Subtitle API (including automatically generated subtitles and subtitle translations)  
  
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BAENLEW8VUJ6G&source=url) [![Build Status](https://travis-ci.org/jdepoix/youtube-transcript-api.svg)](https://travis-ci.org/jdepoix/youtube-transcript-api) [![Coverage Status](https://coveralls.io/repos/github/jdepoix/youtube-transcript-api/badge.svg?branch=master)](https://coveralls.io/github/jdepoix/youtube-transcript-api?branch=master) [![MIT license](http://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat)](http://opensource.org/licenses/MIT) [![image](https://img.shields.io/pypi/v/youtube-transcript-api.svg)](https://pypi.org/project/youtube-transcript-api/) [![image](https://img.shields.io/pypi/pyversions/youtube-transcript-api.svg)](https://pypi.org/project/youtube-transcript-api/)

This is an python API which allows you to get the transcripts/subtitles for a given YouTube video. It also works for automatically generated subtitles, supports translating subtitles and it does not require a headless browser, like other selenium based solutions do!

## Install

It is recommended to [install this module by using pip](https://pypi.org/project/youtube-transcript-api/):

```
pip install youtube_transcript_api
```

If you want to use it from source, you'll have to install the dependencies manually:

```
pip install -r requirements.txt
```

You can either integrate this module [into an existing application](#api), or just use it via an [CLI](#cli).

## API

The easiest way to get a transcript for a given video is to execute:

```python
from youtube_transcript_api import YouTubeTranscriptApi

YouTubeTranscriptApi.get_transcript(video_id)
```

This will return a list of dictionaries looking somewhat like this:

```python
[
    {
        'text': 'Hey there',
        'start': 7.58,
        'duration': 6.13
    },
    {
        'text': 'how are you',
        'start': 14.08,
        'duration': 7.58
    },
    # ...
]
```

You can also add the `languages` param if you want to make sure the transcripts are retrieved in your desired language (it defaults to english).

```python
YouTubeTranscriptApi.get_transcripts(video_ids, languages=['de', 'en'])
```

It's a list of language codes in a descending priority. In this example it will first try to fetch the german transcript (`'de'`) and then fetch the english transcript (`'en'`) if it fails to do so. If you want to find out which languages are available first, [have a look at `list_transcripts()`](#list-available-transcripts)

To get transcripts for a list of video ids you can call:

```python
YouTubeTranscriptApi.get_transcripts(video_ids, languages=['de', 'en'])
```

`languages` also is optional here.

### List available transcripts

If you want to list all transcripts which are available for a given video you can call:

```python
transcript_list = YouTubeTranscriptApi.list_transcripts(video_id, languages=['de', 'en'])
```

This will return a `TranscriptList` object  which is iterable and provides methods to filter the list of transcripts for specific languages and types, like:

```python
transcript = transcript_list.find_transcript(['de', 'en'])
```

By default this module always picks manually created transcripts over automatically created ones, if a transcript in the requested language is available both manually created and generated. The `TranscriptList` allows you to bypass this default behaviour by searching for specific transcript types:

```python
# filter for manually created transcripts
transcript = transcript_list.find_manually_created_transcript(['de', 'en'])

# or automatically generated ones
transcript = transcript_list.find_generated_transcript(['de', 'en'])
```

The methods `find_generated_transcript`, `find_manually_created_transcript`, `find_generated_transcript` return `Transcript` objects. They contain metadata regarding the transcript:

```python
print(
    transcript.video_id,
    transcript.language,
    transcript.language_code,
    # whether it has been manually created or generated by YouTube
    transcript.is_generated,
    # whether this transcript can be translated or not
    transcript.is_translatable,
    # a list of languages the transcript can be translated to
    transcript.translation_languages,
)
```

and provide the method, which allows you to fetch the actual transcript data:

```python
transcript.fetch()
```

### Translate transcript

YouTube has a feature which allows you to automatically translate subtitles. This module also makes it possible to access this feature. To do so `Transcript` objects provide a `translate()` method, which returns a new translated `Transcript` object:

```python
transcript = transcript_list.find_transcript(['en'])
translated_transcript = transcript.translate('de')
print(translated_transcript.fetch())
```

### By example
```python
# retrieve the available transcripts
transcript_list = YouTubeTranscriptApi.get('video_id')

# iterate over all available transcripts
for transcript in transcript_list:

    # the Transcript object provides metadata properties
    print(
        transcript.video_id,
        transcript.language,
        transcript.language_code,
        # whether it has been manually created or generated by YouTube
        transcript.is_generated,
        # whether this transcript can be translated or not
        transcript.is_translatable,
        # a list of languages the transcript can be translated to
        transcript.translation_languages,
    )

    # fetch the actual transcript data
    print(transcript.fetch())

    # translating the transcript will return another transcript object
    print(transcript.translate('en').fetch())
	
# you can also directly filter for the language you are looking for, using the transcript list
transcript = transcript_list.find_transcript(['de', 'en'])  
  
# or just filter for manually created transcripts  
transcript = transcript_list.find_manually_created_transcript(['de', 'en'])  
  
# or automatically generated ones  
transcript = transcript_list.find_generated_transcript(['de', 'en'])
```
  
## CLI  
  
Execute the CLI script using the video ids as parameters and the results will be printed out to the command line:  
  
```  
youtube_transcript_api <first_video_id> <second_video_id> ...  
```  
  
The CLI also gives you the option to provide a list of preferred languages:  
  
```  
youtube_transcript_api <first_video_id> <second_video_id> ... --languages de en  
```

You can also specify if you want to exclude automatically generated or manually created subtitles:

```  
youtube_transcript_api <first_video_id> <second_video_id> ... --languages de en --exclude-generated
youtube_transcript_api <first_video_id> <second_video_id> ... --languages de en --exclude-manually-created
```
  
If you would prefer to write it into a file or pipe it into another application, you can also output the results as json using the following line:  
  
```  
youtube_transcript_api <first_video_id> <second_video_id> ... --languages de en --json > transcripts.json  
```  

Translating transcripts using the CLI is also possible:

```  
youtube_transcript_api <first_video_id> <second_video_id> ... --languages en --translate de
```  

If you are not sure which languages are available for a given video you can call, to list all available transcripts:

```  
youtube_transcript_api --list-transcripts <first_video_id>
```  
  
## Proxy  
  
You can specify a https/http proxy, which will be used during the requests to YouTube:  
  
```python  
from youtube_transcript_api import YouTubeTranscriptApi  
  
YouTubeTranscriptApi.get_transcript(video_id, proxies={"http": "http://user:pass@domain:port", "https": "https://user:pass@domain:port"})  
```  
  
As the `proxies` dict is passed on to the `requests.get(...)` call, it follows the [format used by the requests library](http://docs.python-requests.org/en/master/user/advanced/#proxies).  
  
Using the CLI:  
  
```  
youtube_transcript_api <first_video_id> <second_video_id> --http-proxy http://user:pass@domain:port --https-proxy https://user:pass@domain:port  
```  
## Cookies

Some videos are age restricted, so this module won't be able to access those videos without some sort of authentication. To do this, you will need to have access to the desired video in a browser. Then, you will need to download that pages cookies into a text file. You can use the Chrome extension [cookies.txt](https://chrome.google.com/webstore/detail/cookiestxt/njabckikapfpffapmjgojcnbfjonfjfg?hl=en) or the Firefox extension [cookies.txt](https://addons.mozilla.org/en-US/firefox/addon/cookies-txt/).

Once you have that, you can use it with the module to access age-restricted videos' captions like so. 

```python  
from youtube_transcript_api import YouTubeTranscriptApi  
  
YouTubeTranscriptApi.get_transcript(video_id, cookies=<string of path to your cookies text file>)
  
YouTubeTranscriptApi.get_transcripts([video_id], cookies=<string of path to your cookies text file>)
```  
  
## Warning  
  
 This code uses an undocumented part of the YouTube API, which is called by the YouTube web-client. So there is no guarantee that it won't stop working tomorrow, if they change how things work. I will however do my best to make things working again as soon as possible if that happens. So if it stops working, let me know!  
  
## Donation  
  
If this project makes you happy by reducing your development time, you can make me happy by treating me to a cup of coffee :)  
  
[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BAENLEW8VUJ6G&source=url)
