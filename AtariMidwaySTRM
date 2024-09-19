#!/usr/bin/env python3

import os
import sys
import argparse
import shlex
import subprocess

parser = argparse.ArgumentParser()
parser.add_argument("-i", "--input", required=True, help="input file")
parser.add_argument("-o", "--output", required=True, help="output file")
parser.add_argument("--ffmpeg", help="path to FFmpeg")
parser.add_argument("--ffprobe", help="path to FFprobe")
parser.add_argument("-g", "--game", required=True, help="Game name")
args = parser.parse_args()

header_size = 0x1FF # file length in header is size - 0x1FF, header + padding is 0x1FF long

print('The input audio file (.wav, .mp3, .flac, etc.) will be converted to a 31000 hz signed 16-bit little endian PCM file.\n'
      'The number of channels of the output file depends on the game: Rush 2049 is 6, Gauntlet Dark Legacy is 2, and California Speed/Road Burners is 1.\n'
      'If the input file does not have the correct number of channels, then FFmpeg will upmix/downmix as needed\n'
      'For best results in-game, ensure audio level peak at around 0 dB, otherwise sound effects may drown out music.\n'
      'FFmpeg and FFprobe are required for this script, either automatically found on your $PATH or specified with --ffmpeg and --ffprobe.\n')

if args.ffmpeg is not None:
    try:
        subprocess.run(args.ffmpeg, capture_output=True)
    except FileNotFoundError:
        print("Invalid FFmpeg path.")
        sys.exit(1)
    else:
        ffmpeg_loc = args.ffmpeg
else:
    try:
        subprocess.run('ffmpeg', capture_output=True)
    except FileNotFoundError:
        print("FFmpeg not found in $PATH.")
        sys.exit(1)
    else:
        ffmpeg_loc = "ffmpeg"

if args.ffprobe is not None:
    try:
        subprocess.run(args.ffprobe, capture_output=True)
    except FileNotFoundError:
        print("Invalid FFprobe path.")
        sys.exit(1)
    else:
        ffprobe_loc = args.ffprobe
else:
    try:
        subprocess.run('ffprobe', capture_output=True)
    except FileNotFoundError:
        print("FFprobe not found in $PATH.")
        sys.exit(1)
    else:
        ffprobe_loc = "ffprobe"

if not os.path.exists(args.input):
    print("Input file does not exist.")
    sys.exit(1)

ffprobe_command = ffprobe_loc + " -i " + "\"" + args.input + "\"" + (' -show_entries stream=channels -select_streams'
                                                                     ' a:0 -of compact=p=0:nk=1 -v 0')
num_channels = subprocess.run(shlex.split(ffprobe_command), capture_output=True).stdout
if args.game == "sf2049" or args.game == "sf2049te" or args.game == "sf2049tea" or args.game == "sf2049se" :
    if num_channels == b'1\n':
        ffmpeg_command = [ffmpeg_loc, "-i", args.input, "-filter_complex", r'[0:a]pan=5.1|c0=c0|c1=c0|c4<c0|c2=c0|c3=c0',
                          "-ar", "31000", "-ac", "6", "-f", "s16le", "-acodec", "pcm_s16le", "-y", "temp_unheadered.str"]
    elif num_channels == b'2\n':
        ffmpeg_command = [ffmpeg_loc, "-i", args.input, "-filter_complex", r'[0:a]pan=5.1|c0=c0|c1=c1|c4<c0+c1|c2=c0|c3=c1',
                          "-ar", "31000", "-ac", "6", "-f", "s16le", "-acodec", "pcm_s16le", "-y", "temp_unheadered.str"]
    elif num_channels == b'4\n':
        ffmpeg_command = [ffmpeg_loc, "-i", args.input, "-filter_complex", r'[0:a]pan=5.1|c0=c0|c1=c1|c4<c0+c1+c2+c3|c2=c2|c3=c3',
                          "-ar", "31000", "-ac", "6", "-f", "s16le", "-acodec", "pcm_s16le", "-y", "temp_unheadered.str"]
    else: # just trust what ffmpeg does for the rest
        ffmpeg_command = [ffmpeg_loc, "-i", args.input, "-ar", "31000", "-ac", "6", "-f", "s16le", "-acodec",
                          "pcm_s16le", "-y", "temp_unheadered.str"]
elif args.game == "gauntdl":
    ffmpeg_command = [ffmpeg_loc, "-i", args.input, "-ar", "31000", "-ac", "2", "-f", "s16le", "-acodec",
                      "pcm_s16le", "-y", args.output]
elif args.game == "calspeed" or args.game == "roadburn":
    ffmpeg_command = [ffmpeg_loc, "-i", args.input, "-ar", "31000", "-ac", "1", "-f", "s16le", "-acodec",
                      "pcm_s16le", "-y", args.output]
else:
    print("Unknown game name.")
    exit(1)

subprocess.run(ffmpeg_command)
if args.game == "sf2049" or args.game == "sf2049te" or args.game == "sf2049tea" or args.game == "sf2049se" : # only 2049 seems to have a STRM header, the rest are unheadered
    with open("temp_unheadered.str", 'rb') as pcm_file:
        new_filesize = os.stat("temp_unheadered.str").st_size
        with open(args.output, 'wb') as strm_file:
            strm_file.write(b'STRM')
            strm_file.write(new_filesize.to_bytes(4, byteorder='little'))
            strm_file.write(b'\x36' + (b'\x00' * 3) + b'\x52' + (b'\x00' * (header_size - 12))) # 6 = channels, 0 0 0 = padding, 0x52 = ?
            strm_file.write(pcm_file.read()) # copy rest of file
    os.remove("temp_unheadered.str")
sys.exit(0)
