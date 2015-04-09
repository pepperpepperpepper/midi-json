<pre><div style="font-size: 0.5em;">
NAME
       midi-json - translate MIDI file to a json array of MIDI events

SYNOPSIS
       midi-json [ -u -v ] [ infile [ outfile ] ]

DESCRIPTION
       midi-json  reads  a  standard  MIDI  file  and decodes it into a json array of events, each event with type, track and time parameters. This
       script was adapted from midiscsv by John Walker and preserves all data within the MIDIFILE information in the MIDI file.  midi-json

OPTIONS

       -u        Print how-to-call information.

       -v        Print verbose debugging information on standard error.  The MIDI file header is dumped, along with the length of each track in the
                 file.

FILES
       If no infile is specified or infile is ``-'', midi-json reads its input from standard input; if no outfile is given or outfile is ``-'', CSV
       output is written  to standard output.  The input and output are processed in a strictly serial manner; consequently midi-json may  be  used
       in pipelines without restrictions.

STRUCTURE
       Each object in the json array representation of a MIDI contains at least three fields:

       track     Numeric  field  identifying  the  track to which this record belongs.  Tracks of MIDI data are numbered starting at 1.  Track 0 is
                 reserved for file header, information, and end of file records.

       time      Absolute time, in terms of MIDI clocks, at which this event occurs.  Meta-events for which time is not  meaningful  (for  example,
                 song title, copyright information, etc.) have an absolute time of 0.

       type      Name  identifying  the  type  of  the record.  Record types are text consisting of upper and lower case letters and the underscore
                 (``_''), contain no embedded spaces, and are not enclosed in quotes.

   File Structure Records
       Header,format,nTracks,division...
            The first record of a midi-json file is always the Header record.  Parameters are format: the MIDI file type (0, 1, or 2), nTracks: the
            number of tracks in the file, and division: the number of clock pulses per quarter note.  The Track and Time fields are always zero.

       End_of_file
            The last record in a midi-json file is always an End_of_file record.  Its Track and Time fields are always zero.

       Start_track
            A Start_track record marks the start of a new track, with the Track field giving the track number.  All records between the Start_track
            record and the matching End_track will have the same Track field.

       Track, Time, End_track
            An End_track marks the end of events for the specified Track.  The Time field gives the total duration of  the  track,  which  will  be
            identical to the Time in the last event before the End_track.





    File Meta-Events
       The  following events occur within MIDI tracks and specify various kinds of information and actions.  They may appear at any time within the
       track.  Those which provide general information for which time is not relevant usually appear at the start of the track with Time zero,  but
       this is not a requirement.

       Many of these meta-events include a text string argument.  Text strings are enclosed in ASCII double quote (") characters.  Quote characters
       embedded within strings are represented by two consecutive quotes.  Non-graphic characters in the ISO 8859/1 Latin 1 set  are  output  as  a
       backslash  followed  by  their  three  digit  octal  character  code.  Two consecutive backslashes denote a literal backslash in the string.
       Strings in MIDI files can be extremely long, theoretically as many as 2**28-1 characters; programs which process midi-json files should take
       care  to avoid buffer overflows or truncation resulting from lines containing long string items.  All meta-events which take a text argument
       are identified by a suffix of ``_t''.

       Track, Time, Title_t, "Text"
              The Text specifies the title of the track or sequence.  The first Title meta-event in a type 0 MIDI file, or in the first track of  a
              type 1 file gives the name of the work.  Subsequent Title meta-events in other tracks give the names of those tracks.

       Track, Time, Copyright_t, "Text"
              The Text specifies copyright information for the sequence.  This is usually placed at time 0 of the first track in the sequence.

       Track, Time, Instrument_name_t, "Text"
              The  Text names the instrument intended to play the contents of this track, This is usually placed at time 0 of the track.  Note that
              this meta-event is simply a description; MIDI synthesisers are not required (and rarely if ever) respond to it.  This  meta-event  is
              particularly  useful  in  sequences prepared for synthesisers which do not conform to the General MIDI patch set, as it documents the
              intended instrument for the track when the sequence is used on a synthesiser with a different patch set.

       Track, Time, Marker_t, "Text"
              The Text marks a point in the sequence which occurs at the given Time, for example "Third Movement".

       Track, Time, Cue_point_t, "Text"
              The Text identifies synchronisation point which occurs at the specified Time, for example, "Door slams".

       Track, Time, Lyric_t, "Text"
              The Text gives a lyric intended to be sung at the given Time.  Lyrics are often broken down into  separate  syllables  to  time-align
              them more precisely with the sequence.


       Track, Time, Text_t, "Text"
              This meta-event supplies an arbitrary Text string tagged to the Track and Time.  It can be used for textual information which doesn't
              fall into one of the more specific categories given above.

       Track, 0, Sequence_number, Number
              This meta-event specifies a sequence Number between 0 and 65535, used to arrange multiple tracks in a type 2 MIDI file, or  to  iden-
              tify  the  sequence in which a collection of type 0 or 1 MIDI files should be played.  The Sequence_number meta-event should occur at
              Time zero, at the start of the track.

       Track, Time, MIDI_port, Number
              This meta-event specifies that subsequent events in the Track should be sent to MIDI port (bus) Number,  between  0  and  255.   This
              meta-event  usually appears at the start of a track with Time zero, but may appear within a track should the need arise to change the
              port while the track is being played.

       Track, Time, Channel_prefix, Number
              This meta-event specifies the MIDI channel that subsequent meta-events and System_exclusive events pertain to.   The  channel  Number
              specifies a MIDI channel from 0 to 15.  In fact, the Number may be as large as 255, but the consequences of specifying a channel num-
              ber greater than 15 are undefined.

       Track, Time, Time_signature, Num, Denom, Click, NotesQ
              The time signature, metronome click rate, and number of 32nd notes per MIDI quarter note (24 MIDI  clock  times)  are  given  by  the
              numeric  arguments.  Num gives the numerator of the time signature as specified on sheet music.  Denom specifies the denominator as a
              negative power of two, for example 2 for a quarter note, 3 for an eighth note, etc.  Click  gives  the  number  of  MIDI  clocks  per
              metronome click, and NotesQ the number of 32nd notes in the nominal MIDI quarter note time of 24 clocks (8 for the default MIDI quar-
              ter note definition).

       Track, Time, Key_signature, Key, Major/Minor
              The key signature is specified by the numeric Key value, which is 0 for the key of C, a positive value for each sharp above C,  or  a
              negative  value  for  each flat below C, thus in the inclusive range -7 to 7.  The Major/Minor field is a quoted string which will be
              "major" for a major key and "minor" for a minor key.

       Track, Time, Tempo, Number
              The tempo is specified as the Number of microseconds per quarter note, between 1 and 16777215.  A value of 500000 corresponds to  120
              quarter  notes ("beats") per minute.  To convert beats per minute to a Tempo value, take the quotient from dividing 60,000,000 by the
              beats per minute.

       Track, 0, SMPTE_offset, Hour, Minute, Second, Frame, FracFrame
              This meta-event, which must occur with a zero Time at the start of a track, specifies the SMPTE time code at which  it  should  start
              playing.  The FracFrame field gives the fractional frame time (0 to 99).

       Track, Time, Sequencer_specific, Length, Data, ...
              The Sequencer_specific meta-event is used to store vendor-proprietary data in a MIDI file.  The Length can be any value between 0 and
              2**28-1, specifying the number of Data bytes (between 0 and 255) which follow.  Sequencer_specific records may be very long; programs
              which process midi-json files should be careful to protect against buffer overflows and truncation of these records.

       Track, Time, Unknown_meta_event, Type, Length, Data, ...
              If  midi-json  encounters  a  meta-event with a code not defined by the standard MIDI file specification, it outputs an unknown meta-
              event record in which Type gives the numeric meta-event type code, Length the number of data bytes in the meta-event,  which  can  be
              any  value  between  0  and 2**28-1, followed by the Data bytes.  Since meta-events include their own length, it is possible to parse
              them even if their type and meaning are unknown.

   Channel Events
       These events are the "meat and potatoes" of MIDI files: the actual notes and modifiers that command the instruments  to  play  the  music.
       Each  has  a  MIDI  channel  number as its first argument, followed by event-specific parameters.  To permit programs which process the json
       files to easily distinguish them from meta-events, names of channel events all have a suffix of ``_c''.

       Track, Time, Note_on_c, Channel, Note, Velocity
              Send a command to play the specified Note (Middle C is defined as Note number 60; all other notes are relative in the MIDI specifica-
              tion, but most instruments conform to the well-tempered scale) on the given Channel with Velocity (0 to 127).  A Note_on_c event with
              Velocity zero is equivalent to a Note_off_c.

       Track, Time, Note_off_c, Channel, Note, Velocity
              Stop playing the specified Note on the given Channel.  The Velocity should be zero, but you never know what you'll  find  in  a  MIDI
              file.

       Track, Time, Pitch_bend_c, Channel, Value
              Send  a  pitch bend command of the specified Value to the given Channel.  The pitch bend Value is a 14 bit unsigned integer and hence
              must be in the inclusive range from 0 to 16383.

       Track, Time, Control_c, Channel, Control_num, Value
              Set the controller Control_num on the given Channel to the specified Value.  Control_num and Value must be in the inclusive  range  0
              to  127.   The  assignment  of  Control_num  values to effects differs from instrument to instrument.  The General MIDI specification
              defines the meaning of controllers 1 (modulation), 7 (volume), 10 (pan), 11 (expression), and 64 (sustain), but not  all  instruments
              and patches respond to these controllers.  Instruments which support those capabilities usually assign reverberation to controller 91
              and chorus to controller 93.


       Track, Time, Program_c, Channel, Program_num
              Switch the specified Channel to program (patch) Program_num, which must be between 0 and 127.  The program  or  patch  selects  which
              instrument and associated settings that channel will emulate.  The General MIDI specification provides a standard set of instruments,
              but synthesisers are free to implement other sets of instruments and many permit the user to create custom patches and assign them to
              program numbers.

              Apparently due to instrument manufacturers' skepticism about musicians' ability to cope with the number zero, many instruments number
              patches from 1 to 128 rather than the 0 to 127 used within MIDI files.  When interpreting Program_num values, note that they  may  be
              one less than the patch numbers given in an instrument's documentation.

       Track, Time, Channel_aftertouch_c, Channel, Value
              When a key is held down after being pressed, some synthesisers send the pressure, repeatedly if it varies, until the key is released,
              but do not distinguish pressure on different keys played simultaneously and held down.  This is  referred  to  as  ``monophonic''  or
              ``channel''  aftertouch  (the  latter  indicating it applies to the Channel as a whole, not individual note numbers on that channel).
              The pressure Value (0 to 127) is typically taken to apply to the last note played, but instruments are not guaranteed  to  behave  in
              this manner.

       Track, Time, Poly_aftertouch_c, Channel, Note, Value
              Polyphonic  synthesisers  (those  capable  of  playing  multiple notes simultaneously on a single channel), often provide independent
              aftertouch for each note.  This event specifies the aftertouch pressure Value (0 to 127) for the specified Note on the given Channel.

   System Exclusive Events
       System Exclusive events permit storing vendor-specific information to be transmitted to that vendor's products.

       Track, Time, System_exclusive, Length, Data, ...
              The  Length  bytes  of  Data  (0 to 255) are sent at the specified Time to the MIDI channel defined by the most recent Channel_prefix
              event on the Track, as a System Exclusive message.  Note that Length can be any value between 0 and 2**28-1.  Programs which  process
              midi-json files should be careful to protect against buffer overflows and truncation of these records.

       Track, Time, System_exclusive_packet, Length, Data, ...
              The  Length  bytes  of  Data  (0 to 255) are sent at the specified Time to the MIDI channel defined by the most recent Channel_prefix
              event on the Track.  The Data bytes are simply blasted out to the MIDI bus without any prefix.  This message is used by MIDI  devices
              which  break  up  long  system  exclusive  message into small packets, spaced out in time to avoid overdriving their modest microcon-
              trollers.  Note that Length can be any value between 0 and 2**28-1.  Programs which process midi-json files should be careful to pro-
              tect against buffer overflows and truncation of these records.




BUGS
       midi-json  assumes its input is a well-formed standard MIDI file; while some error checking is performed, gross errors in the input file may
       cause midi-json to crash.

       Please report problems to bugs@spermwhale.info.


AUTHOR
       midi-json is by Pube Douchevitz: http://www.spermwhale.info/
       
       original midicsv is by John Walker: http://www.fourmilab.ch/

       This software is in the public domain.  Permission to use, copy, modify, and distribute this software and its documentation for any  purpose
       and  without  fee is hereby granted, without any conditions or restrictions.  This software is provided ``as is'' without express or implied
       warranty.

4th Berkeley Distribution                                            4 JUL 2014                                                        MIDI-JSON(1)
</div></pre>
