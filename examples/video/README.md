#### How does video streaming work?

* The communication of DJI Ryze Tello is done utilizing IEEE 802.11 frames in an IP-based manner. All data for live streaming is sent within the payload of 802.11 QoS data frames (type 2, subtype 8) by UDP to the destination port 77x7.
* The drone splits video data into logical chunks.
  * A chunk might be sent in one piece, thus by just one 802.11 frame. Due to the limit of size for the payload in 802.11 frames to avoid collisions, a chunk might be fragmented in the 802.11 layer and sent piece by piece, thus by several 802.11 frames.
  * The first piece of any chunk contains two bytes in addition to the video data. These bytes represent two integers which indicate the sequence of parts and pieces and a flag highlighting the last piece.
    In an application, this integers can be used for an ordered reassembly of the video stream, which is a more robust way than reassembly using the sequence and fragment numbers provided by the 802.11 layer.
* Since 802.11 provides error detection by cyclic redundancy check (CRC), 4 bytes in the payload are used to constitute the frame check sequence (FCS). The FCS is not part of the video data.

#### Where to find video data?

* The format of the video stream is yuv420p at a resolution of 960x720 pixels. For compression prior transmission, h.264 is used to encode the video stream.

* To get one h.264 video frame three ingredients - or NALs in terms of h.264 - are needed:

  1. sequence parameter set (SPS),
  2. picture parameter set (PPS),
  3. keyframe. A keyframe is a slice layer without partitioning instantaneous decoding refresh (non-IDR).

  Hence NALs expose unique start prefix codes, they can easily be found in the payload.

* The start prefix codes are as follows:

  1. SPS = 0x00000167,
  2. PPS = 0x00000168,
  3. keyframe = 0x00000165.

  However, the 6 in each start prefix code could become another digit.

#### How to reassemble video frames?

* All bytes of consecutive NALs have to be put together to yield an h.264 frame. Therefore, the fragmentation done in the 802.11 layer for any keyframe has to be reversed.

* The following shows the size of an h.264 frame with 1 SPS, 1 PPS and 1 keyframe which is composed of n pieces each split into 3 fragments:

    * 0x00000167 .. [26 bytes]
    * 0x00000168 .. [16 bytes]
    * 0x00000165 .. [approx. (n * (964 + 1040 + 912)) bytes]

    Nevertheless, regularly the very last fragment is smaller than 912 bytes and usually, the last piece consists of less than 3 fragments.

#### How to decode video frames?

* Piping the video data to an external process is assumed to be the most common use case for decoding.
  FFmpeg (and Libav) is well suited to decode h.264 frames through pipes.
* The following command shows an example using FFmpeg for decoding.
  Reassembled video data feed to stdin will be decoded, converted to RGB and issued on stdout.
    * ffmpeg -i - -f image2pipe -pix_fmt rgb24 -vcodec rawvideo -
* In comparison to yuv420p, the size of an RGB frame is doubled to (960 * 720 * 3) bytes. Nevertheless, further processing of RGB data is widely supported and certainly more simple than dealing with other formats.

#### How to capture video streams?

* The log file in this example has been created with [Tepsots](https://github.com/m6c7l/tepsots), a simple IEEE 802.11 packet sniffer.
* You can sniff non-encrypted video streams with a computer having a wireless 2.4 GHz network device placed into monitoring mode from any Tello drone in the vicinity.

