# atari-midway-strm
A script for converting audio files to Atari/Midway's arcade STRM format. This format was used in multiple arcade games such as San Francisco Rush 2049, Road Burners, California Speed, and Gauntlet Dark Legacy. The files slightly differ between games but they all consist of 31250Hz signed 16-bit little-endian PCM audio. 

The number of audio channels depends on the game; Road Burners and California Speed use 1 channel, Gauntlet Dark Legacy uses 2, and Rush 2049 uses 6 (5 channel audio with empty last channel). Rush 2049's files also contain a header not seen in the other games, so this script will also generate that header for the files that need it.

**Note**: This script requires FFmpeg and FFprobe to be installed at the $PATH, or their paths can be specified with parameters.

Example usage:
```
./AtariMidwaySTRM.py -i raverush.wav -o raverush.str --game sf2049
```
