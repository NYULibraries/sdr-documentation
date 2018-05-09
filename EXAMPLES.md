# SDR Example/Helpful Commands
Stephen Balogh <sgb334@nyu.edu>

## Unix tools

Below are glosses on some particularly useful Unix command-line tools for working with collections. I won't go much into any of the basic ones, since there are many general resources online, but in addition to those listed below you should probably have some knowledge of: `pwd`, `ls`, `cd`, `mkdir`, `rm` (and why you should be careful with it!), `touch`, `less` (or some other scrolling text reader), `cat`, `vim` (or `emacs` or `nano` or whatever), `ssh`...

### Find
`find` is great! It does a lot, but most important for us is that it allows you to recursively search through a directory to list all of the files that exist in it or any of its subdirectories.

```bash
## Find all files on your desktop
cd ~/Desktop
find . ## the . just means current directory
```

Note that you can write the output of the above to a text file, instead of to the console:

```bash
find . > ~/Desktop/find_result.txt ## Creates a new file containing the results
```

You can also add name restictions to `find`; for instance, maybe you want to gather a list of all files within a directory (*and* any of its subdirectories) which are ESRI Shapefiles. Therefore, we are only interested in files ending in `.shp`:

```bash
cd east_view_collection
# this assumes that there's a folder called east_view_collection that has a bunch of shapefiles in it
find . -name "*.shp"
# this recursively locates all files matching the given pattern
find . -name "*.shp" > ~/Desktop/east_view_files.txt
# this saves the result to a text file, which makes it easy to see how many shapefiles exist
```
### Grep

`grep` is a very powerful way to search through a stream of text.

TODO: provide examples

### Zip

This is just the command-line tool for creating ZIP archives.

```bash
cd ~/Desktop/sdr_staging
### Assume I have a folder called `nyu_1234_56789` in this folder

zip -r nyu_1234_56789.zip ./nyu_1234_56789/
```
The `-r` flag means that the zip operation should be recursive; therefore, if you are zipping a directory, it just means that you want the zip file to contain all of the contents of that directory. This is not needed if you are just zipping a single file, like an individual `.tiff`.

### Curl

`curl` is a very widely used utility for making HTTP(S) requests to websites. It's particularly handy for connecting to web (REST) APIs.

```bash
## A simple GET request for a JSON page
curl https://geo.nyu.edu/catalog/nyu_2451_36737.json
```

If you have `jq`, the JSON processor, installed on your system, it can be used in conjunction with `curl`:


```bash
## GET some JSON and pipe it into a JSON processor
curl https://geo.nyu.edu/catalog/nyu_2451_36737.json | jq
## Now you should see a prettified and syntax-highlighted version of the JSON object!
```

GET commands are great, but `curl` can also be used for any of the other HTTP verbs (PUT, POST, DELETE). Actually, `curl` is used as the back-end for many SdrFriend actions that involve communicating with a web API, so if you ever need to debug something, you can try prying into the `SdrFriend` source code and reconstructing / modying the original `curl` command.

By the way, the same UNIX "write-to-file" logic applies with `curl`:

```bash
curl https://geo.nyu.edu/catalog/nyu_2451_36737.json > ~/Desktop/nyu_2451_36737.json
```
