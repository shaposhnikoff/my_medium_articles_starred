This is a collection of my medium starred articles

You can create your own collection of such articles , step by step instruction below

1. Download you bookmarks archive
2. Put it to bookmarks folder
3. Run the 


find bookmarks/ -type f \
    | xargs grep -hEo 'https?:\/\/[=a-zA-Z0-9\_\/\?\&\.\-]+' \
    | sed '/Binary/d' \
    | sort \
    | uniq > urlin.txt
