# SDR Example/Helpful Commands
Stephen Balogh <sgb334@nyu.edu>

## Unix tools

### Find
`find` 
```bash
cd east_view_collection
# this assumes that there's a folder called east_view_collection that has a bunch of shapefiles in it
find . -name "*.shp"
# this recursively locates all files matching the given pattern
find . -name "*.shp" > ~/Desktop/east_view_files.txt
# this saves the result to a text file, which makes it easy to see how many shapefiles exist
```
