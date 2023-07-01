# react-native-vlc-media-player

## Supported RN Versions

0.59 > 0.62 and up
PODs are updated to works with 0.61 and up.(Tested in 0.61.5 and 0.62 and 0.63)

## Supported formats

Support for network streams, RTSP, RTP, RTMP, HLS, MMS.
Play all files,[ in all formats, including exotic ones, like the classic VLC media player.](#-More-formats)
Play MKV, multiple audio tracks (including 5.1), and subtitles tracks (including SSA!)

### Add it to your project

Run

`npm i liquid-player`

## android

Should work without any specific settings

gradle build might failed with `More than one file was found with OS independent path 'lib/x86/libc++_shared.so'` error. If that happens add the following block to your `android/app/build.gradle`

```
tasks.whenTaskAdded((tas -> {
    // when task is 'mergeLocalDebugNativeLibs' or 'mergeLocalReleaseNativeLibs'
    if (tas.name.contains("merge") && tas.name.contains("NativeLibs")) {
        tasks.named(tas.name) {it
            doFirst {
                java.nio.file.Path notNeededDirectory = it.externalLibNativeLibs
                        .getFiles()
                        .stream()
                        // for React Native 0.71, the file value now contains "jetified-react-android" instead of "jetified-react-native"
                        .filter(file -> file.toString().contains("jetified-react-native"))
                        .findAny()
                        .orElse(null)
                        .toPath();
                Files.walk(notNeededDirectory).forEach(file -> {
                    if (file.toString().contains("libc++_shared.so")) {
                        Files.delete(file);
                    }
                });
            }
        }
    }
}))
```

Explain: react-native and libvlc both import `libc++_shared.so`, but we cannot use `packagingOptions.pickFirst` to handle this case, because libvlc-all:3.6.0-eap5 will crash when using `libc++_shared.so`, so we have to use `libc++_shared.so` from libvlc, so I add a gradle script to delete `libc++_shared.so` from react-native to solve this.

Reference: https://stackoverflow.com/questions/74258902/how-to-define-which-so-file-to-use-in-gradle-packaging-options

Explain why we have to use libvlc-all:3.6.0-eap5 instead of libvlc-all:3.2.6: libvlc-all:3.2.6 has a bug that subtitle won't display on Android 12 and 13, so we have to upgrade libvlc to support subtitle display on Android 12 and 13.

Reference: https://code.videolan.org/videolan/vlc-android/-/issues/2252

## Use

```
import LiquidPlayer from 'liquid-player';

<LiquidPlayer
    style={[styles.video]}
    videoAspectRatio="16:9"
    source={{ uri: "https://www.radiantmediaplayer.com/media/big-buck-bunny-360p.mp4"}}
/>
```

### LiquidPlayer Props

| Prop                | Description                                                                                                                                        | Default |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| `source`            | Object that contains the uri of a video or song to play eg `{{ uri: "https://video.com/example.mkv" }}`                                            | `{}`    |
| `subtitleUri`       | local subtitle file path，if you want to hide subtitle, you can set this to an empty subtitle file，current we don't support a `hide subtitle` prop. |         |
| `paused`            | Set to `true` or `false` to pause or play the media                                                                                                | `false` |
| `repeat`            | Set to `true` or `false` to loop the media                                                                                                         | `false` |
| `rate`              | Set the playback rate of the player                                                                                                                | `1`     |
| `seek`              | Set position to seek between `0` and `1` (`0` being the start, `1` being the end , use `position` from the progress object )                       |         |
| `volume`            | Set the volume of the player (`number`)                                                                                                            |         |
| `muted`             | Set to `true` or `false` to mute the player                                                                                                        | `false` |
| `audioTrack`        | Set audioTrack id (`number`)   (see `onLoad` callback VideoInfo.audioTracks)                                                                       |         |
| `textTrack`         | Set textTrack(subtitle) id  (`number`)   (see `onLoad` callback- VideoInfo.textTracks)                                                             |         |                      
| `playInBackground`  | Set to `true` or `false` to allow playing in the background                                                                                        | false   |
| `videoAspectRatio ` | Set the video aspect ratio eg `"16:9"`                                                                                                             |         |
| `autoAspectRatio`   | Set to `true` or `false` to enable auto aspect ratio                                                                                               | false   |
| `resizeMode`        | Set the behavior for the video size (`fill, contain, cover, none, scale-down`)                                                                     | none    |
| `style`             | React native stylesheet styles                                                                                                                     | `{}`    |

#### Callback props

Callback props take a function that gets fired on various player events:

| Prop           | Description                                                                                                                                                                                                          |
|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `onPlaying`    | Called when media starts playing returns eg `{target: 9, duration: 99750, seekable: true}`                                                                                                                           |
| `onProgress`   | Callback containing `position` as a fraction, and `duration`, `currentTime` and `remainingTime` in seconds <br />&nbsp; ◦ &nbsp;eg `{  duration: 99750, position: 0.30, currentTime: 30154, remainingTime: -69594 }` |
| `onPaused`     | Called when media is paused                                                                                                                                                                                          |
| `onStopped `   | Called when media is stoped                                                                                                                                                                                          |
| `onBuffering ` | Called when media is buffering                                                                                                                                                                                       |
| `onEnded`      | Called when media playing ends                                                                                                                                                                                       |
| `onError`      | Called when an error occurs whilst attempting to play media                                                                                                                                                          |
| `onLoad`       | Called when video info is loaded, Callback containing VideoInfo                                                                                                                                                      |

VideoInfo example:

```
 {
    duration: 30906, 
    videoSize: {height: 240, width: 32},
    audioTracks: [
            {id: -1, name: "Disable"}, 
            {id: 1, name: "Track 1"}, 
            {id: 3, name: "Japanese Audio (2ch LC-AAC) - [Japanese]"}
    ], 
    textTracks: [
        {id: -1, name: "Disable"}, 
        {id: 4, name: "Track 1 - [English]"}, 
        {id: 5, name: "Track 2 - [Japanese]"}
    ],    
}
```

## More formats

Container formats: 3GP, ASF, AVI, DVR-MS, FLV, Matroska (MKV), MIDI, QuickTime File Format, MP4, Ogg, OGM, WAV, MPEG-2 (ES, PS, TS, PVA, MP3), AIFF, Raw audio, Raw DV, MXF, VOB, RM, Blu-ray, DVD-Video, VCD, SVCD, CD Audio, DVB, HEIF, AVIF
Audio coding formats: AAC, AC3, ALAC, AMR, DTS, DV Audio, XM, FLAC, It, MACE, MOD, Monkey's Audio, MP3, Opus, PLS, QCP, QDM2/QDMC, RealAudio, Speex, Screamtracker 3/S3M, TTA, Vorbis, WavPack, WMA (WMA 1/2, WMA 3 partially).
Capture devices: Video4Linux (on Linux), DirectShow (on Windows), Desktop (screencast), Digital TV (DVB-C, DVB-S, DVB-T, DVB-S2, DVB-T2, ATSC, Clear QAM)
Network protocols: FTP, HTTP, MMS, RSS/Atom, RTMP, RTP (unicast or multicast), RTSP, UDP, Sat-IP, Smooth Streaming
Network streaming formats: Apple HLS, Flash RTMP, MPEG-DASH, MPEG Transport Stream, RTP/RTSP ISMA/3GPP PSS, Windows Media MMS
Subtitles: Advanced SubStation Alpha, Closed Captions, DVB, DVD-Video, MPEG-4 Timed Text, MPL2, OGM, SubStation Alpha, SubRip, SVCD, Teletext, Text file, VobSub, WebVTT, TTML
Video coding formats: Cinepak, Dirac, DV, H.263, H.264/MPEG-4 AVC, H.265/MPEG HEVC, AV1, HuffYUV, Indeo 3, MJPEG, MPEG-1, MPEG-2, MPEG-4 Part 2, RealVideo 3&4, Sorenson, Theora, VC-1,[h] VP5, VP6, VP8, VP9, DNxHD, ProRes and some WMV.

## Example use with react-native-fs
- Path param req
- Only PNG or JPEG
- Can not overwrite existing file, you should use react-native-fs to managing files
- May need set `<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />` in AndroidManifes.xml (propably only for external storage path)

```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 *
 * @format
 * @flow strict-local
 */

import React, {useRef, useState} from 'react';
import {
  SafeAreaView,
  ScrollView,
  StatusBar,
  StyleSheet,
  Text,
  useColorScheme,
  View,
    Button,
    Image
} from 'react-native';

import {
  Colors,
  Header,
} from 'react-native/Libraries/NewAppScreen';
import LiquidPlayer from 'liquid-player';
import RNFS from 'react-native-fs'


const App= () => {
  const isDarkMode = useColorScheme() === 'dark';
  const video = useRef();
  const [path,setPath] = useState(null);

  const backgroundStyle = {
    backgroundColor: isDarkMode ? Colors.darker : Colors.lighter,
  };

  return (
    <SafeAreaView style={backgroundStyle}>
      <StatusBar
        barStyle={isDarkMode ? 'light-content' : 'dark-content'}
        backgroundColor={backgroundStyle.backgroundColor}
      />
      <ScrollView
        contentInsetAdjustmentBehavior="automatic"
        style={backgroundStyle}>
        <Header />
        <View
          style={{
            backgroundColor: isDarkMode ? Colors.black : Colors.white,
          }}>
          <LiquidPlayer
              ref={video}
              style={{width: '100%', height: 400}}
              videoAspectRatio="16:9"
              onSnapshot={(e) => {
                console.log(e.nativeEvent);
                if(e.nativeEvent.isSuccess) {
                  setPath(e.nativeEvent.path);
                }
                }
          }
              onError={(e) => console.log(e)}
              paused={false}
              source={{ uri: "rtsp://zephyr.rtsp.stream/movie?streamKey=0eb010a968a956a14b49101d43342bed"}}
          />
          <Button onPress={() => {
            video.current.snapshot(RNFS.DocumentDirectoryPath + '/obraz.png');
          }} title={"SnapShot!"} />
        </View>
        <Text>{path || 'nie ma patha'}</Text>
        {path !== null &&
        <Image source={{uri: 'file://' + path}} style={{width: '100%', height: 400}} />
        }
      </ScrollView>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  sectionContainer: {
    marginTop: 32,
    paddingHorizontal: 24,
  },
  sectionTitle: {
    fontSize: 24,
    fontWeight: '600',
  },
  sectionDescription: {
    marginTop: 8,
    fontSize: 18,
    fontWeight: '400',
  },
  highlight: {
    fontWeight: '700',
  },
});

export default App;
```
