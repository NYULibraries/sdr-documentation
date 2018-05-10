# Code Snippits
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
