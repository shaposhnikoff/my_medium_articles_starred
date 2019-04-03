
# Adding chapters to an MP4 file using ffmpeg

I have some music videos stored locally, and a couple of them have long cinematic intros — in one case, two and a half minutes long, on a song that’s only three minutes long. We use the music videos for dance time with my kiddo before bed, and we’re really only interested in the music. So I considered removing the cinematic intro entirely, but decided it’d be more fun to do something more sophisticated.

So, I found [https://ffmpeg.org/ffmpeg-formats.html#Metadata-1](https://ffmpeg.org/ffmpeg-formats.html#Metadata-1), which describes exactly what to do. I timed out the different chapters I wanted to created, and added them to the format described above. I got this file:

    ;FFMETADATA1
    title=Havana
    ;this is a comment
    artist=Camila Cabello

    [CHAPTER]
    TIMEBASE=1/1000
    START=0
    #chapter ends at 02:28
    END=147999
    title=Intro

    [CHAPTER]
    TIMEBASE=1/1000
    #chapter starts at 02:28
    START=148000
    #chapter ends at 05:22
    END=321999
    title=Music

    [CHAPTER]
    TIMEBASE=1/1000
    #chapter starts at 05:22
    START=322000
    #end of the file
    END=402000
    title=Outro

    [STREAM]
    title=Havana

and named it metadata .

Then it couldn’t be simpler: ffmpeg -i /data/havana.mp4 -i metadata -map_metadata 1 -codec copy new_havana.mp4

And now I can skip straight to the music when it’s dance time, but can still watch the entire video (it’s pretty funny, I have to say) if I choose.
