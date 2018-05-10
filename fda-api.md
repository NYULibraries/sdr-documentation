# Faculty Digital Archive REST API Primer
Stephen Balogh <sgb334@nyu.edu>, 9/28/2017

I’ve compiled some instructions on how to do some simple actions using the Faculty Digital Archive REST API, using `curl` as an example. `Curl` is a command-line tool that is widely available, and should be already installed on any Mac (just type these commands into a Terminal session).

The API is the one defined by DSpace, which is documented here: https://wiki.duraspace.org/display/DSDOC5x/REST+API

You can explore the REST routes just by going here in your browser: https://archive.nyu.edu/rest/

The examples below walk through the process of getting authenticated to use the API (grabbing a token), creating a new FDA record, and adding content to that record. Other actions, like adding metadata fields, deleting bitstreams, deleting records, and so forth are documented in the wiki page linked above.

*Note that all of the example `curl` commands below are meant to be executed as a single instruction, even though many are long and wrap around multiple lines in this document.*

## Getting an authentication token

Essentially, what you do is POST your FDA username and password to the API (securely, as it’s over HTTPS), and get back a token in return. This token is valid for some period of time, and needs to be present in subsequent API interactions in order to identify you.

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"email":"username@nyu.edu","password":"this-is-my-password"}' \
  https://archive.nyu.edu/rest/login
```

This will return an alphanumeric string that serves as your token for future requests during your session (e.g. `abcd1234-abcd-1234-abcd-abcd1234abcd`). Hold on to this.

## Creating a record

Creating a record requires that you POST a record object to the collection where you want it to reside. A record object consists of an array of metadata elements, all pertaining to the same desired record. Here’s a very simple example of a record object with two metadata fields (title and contributor):

```json
[
  {
    "key": "dc.title",
    "language": "en_US",
    "value": "Whatever"
  },
  {
    "key": "dc.contributor.author",
    "language": "en_US",
    "value": "Jane Smith"
  }
]
```

So basically, each metadata element object in the array contains the DSpace-specific key which identifies the relevant field, a “value” for this key, and a language reference that describes the language of your value. It can be tricky to figure out the desired keys –– one strategy is to just look at metadata of existing records via the API: see [this record](https://archive.nyu.edu/rest/items/39592/metadata) as an example of one.

This is what it looks like to POST such an object:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "rest-dspace-token: abcd1234-abcd-1234-abcd-abcd1234abcd" \
  -d '{"metadata" : [{"key":"dc.title","language":"en_US","value":"Whatever"},{"key":"dc.contributor.author","language":"en_US","value":"Jane Smith"}]}' \
  https://archive.nyu.edu/rest/collections/123/items
```

In the above example, `abcd1234-abcd-1234-abcd-abcd1234abcd` is just the token that you received in Step 1; the next bolded object is the JSON-encoded metadata; and the last thing to pay attention to is the collection ID (`123`), which identifies the collection which will contain the resulting record.

If everything is successful, you should get back (in JSON) an object similar to this:

```json
{
  "id": 40894,
  "name": "Whatever",
  "handle": "2451/39874",
  "type": "item",
  "link": "/rest/items/40894",
  "expand": [
    "metadata",
    "parentCollection",
    "parentCollectionList",
    "parentCommunityList",
    "bitstreams",
    "all"
  ],
  "lastModified": "2017-08-28 16:11:02.611",
  "parentCollection": null,
  "parentCollectionList": null,
  "parentCommunityList": null,
  "bitstreams": null,
  "archived": "true",
  "withdrawn": "false"
}
```

This is useful to hold on to, since it contains the internal DSpace ID for the record that was created, as well as the Handle ID. Confusingly, these are not the same. You’ll need the DSpace ID for any API call that refers to the record, such as uploading a bitstream.

## Uploading a bitstream to a record

To upload a bitstream (in other words, an actual file which will be a part of your containing record), you POST a binary file to an existing record. As far as I am aware, there are no limits on max filesize (unlike with the web UI).

```bash
curl --data-binary "@/path/to/file.zip" \
  -H "Accept: application/json" \
  -H "rest-dspace-token: abcd1234-abcd-1234-abcd-abcd1234abcd" \
  https://archive.nyu.edu/rest/items/12345/bitstreams?name=file.zip
```

In this example, `/path/to/file.zip` is any file path on your personal computer to the document you want to upload, `12345` is the DSpace identifier of the record to which you want to add the bitstream, and `file.zip` at the end of the URL is the way that you specify the "name" attribute of the bitstream (you need to set this, but most of the time you probably want it to just be the same as the filename on your system).
