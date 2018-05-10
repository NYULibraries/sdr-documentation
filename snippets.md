# Code Snippets
Stephen Balogh <sgb334@nyu.edu>

Why waste when you can reuse?

### Unzipping nested .zip files, then deleting the .zip

```ruby
require 'find'

FOLDER_WITH_CONTAINERS="/Users/andrewbattista/Desktop/sdr_collection"

## Gather all paths, recursively, from the folder above, but only
## retain those ending with `.zip`
zip_paths = Find.find(FOLDER_WITH_CONTAINERS).select{ |path| File.extname(path) == ".zip"}

## For each zip path, we determine its parent directory, and then
## run bash commands to unzip and delete
zip_paths.each do |zp|
  containing_directory = zp.gsub(File.basename(zp),"")
  `cd #{containing_directory}; unzip #{zp}; rm #{zp}`
end
```

## General Ruby Tricks

#### Turn off Ruby interpreter "echo"
```ruby
irb_context.echo = false
```

#### Consume a list in batches
```ruby
some_list = (1..10000).to_a ## makes a list from the range 1 -> 10000
some_list.each_slice(100) do |slice| ## iterates over the list in slices of size 100
  puts slice[0] ## print the first element of each slice/batch
end
```
