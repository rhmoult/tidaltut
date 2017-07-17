# tidaltut

## My own Tidal Cycles Tutorial

### maybe this can be useful for someone some day. Atm, it is only to have organized what I have been learning in Tidal.

## Cheat Sheet

## Slack Excerpts


averageaht 5:43 PM
  I know this information can be put together from the tutorial, but I can't figure it out: 
  How can I play a chord progression that lasts more than a single cycle (say, 6 quarter notes) every N cycles?

  assume 4/4 time. every 4 measures I want to play some notes, say " `c1 e1 g1 b1 d1` ", all quarter notes
  let me know if that's clear or not
  I know I can pitch notes with up or use midinotes to create the notes
  I can get something to happen every 4 measures using every 4, but this seems like a hack:
  `d1 $ every 4 (const $ sound "bd") $ sound "~"`
  guess I use slowcat to create a pattern longer than a single cycle

efairbanks 9:11 PM
  You can use slow and fast to change the speed of a Pattern, not just ParamPatterns.

yaxu 9:12 PM
  is this what you want? `"<[c1 e1 g1 b1] [d1 ~ ~ ~] ~ ~>"`

efairbanks 9:13 PM
  `d1 $ s "pluck*2" # up (slow 8 "0 3 5 7")`
  Or even
  `d1 $ s "pluck*2" |*| up "[0,3,7]" |*| up (slow 8 "0 3 5 7")
If you want to get more advanced, you can apply a slow progression to multiple elements:
do
  cps 1
  let progression p = p |*| up (slow 8 $ "0 5 10 7")
  let melody = progression $ stut 4 0.3 1.033 $ fast 2 $ up "0 3 7 10" |*| up "12" # s "pluck"
  let bass = progression $ s "pluck" |*| speed "0.5" |*| gain "1.0" # shape 0.6 # cut "-1"
  d1 $ stack [melody, bass]

averageaht 9:16 PM
@yaxu yes, that is what I want, can I slow that down so that it happens in 2 cycles I guess?

averageaht 9:17 PM
@efairbanks I dont get why the plucks are playing twice in your example, even if I remove *2
oh, I guess 8/4=2

averageaht 9:18 PM
d1 $ s "pluck" # up (slow 6 "0 3 5 7 9 11")  works

efairbanks 9:19 PM
Yeah, and there slow is essentially changing the length of a cycle.
So you could alternate between two melodies per cycle like so:
d1 $ s "pluck" # up (slow 6 "<[0 3 5 7 9 11] [2 4 8 11 13 15]>")

averageaht 9:20 PM
hmm I don't want to change the length of the cycle

efairbanks 9:21 PM
It's only changing it local to the Pattern, not globally.

averageaht 9:21 PM
how do I get something that's "in time" with my other patterns?

efairbanks 9:21 PM
Another way to think of slow is that it slows down the time that the pattern is receiving, so to speak.
The pattern: "<[0 3 5 7 9 11] [2 4 8 11 13 15]>" is operating on a different time scale than the "pluck" pattern because you've slowed it.
But you could then do.
d1 $ (s "pluck" # up (slow 6 "<[0 3 5 7 9 11] [2 4 8 11 13 15]>")) |*| up "0 7 12"
And the cycle "0 7 12" is being evaluated after all that, so it is operating at the normal time scale and will occupy one normal cycle.

efairbanks 9:25 PM
I mean, not strictly. There are a lot of ways to achieve what you want to do. The <> mean "every cycle play the next element within <>"
Where each element occupies one cycle.
And then the whole thing is being slowed by a factor of 6.

averageaht 9:26 PM
okay, so there are six notes in those two cycles


efairbanks
Yeah.
6 notes in one cycle, 6 notes in the next.
And because the whole thing is slowed by a factor of 6 after that, each cycle is 6 times as long as it normally would be.
So each note ends up being the length of "one cycle" from a global reference point.

averageaht 9:27 PM
so I guess the alternative to <> is slowcat?

yaxu 9:28 PM
an important point is that tidal has no fixed idea of what a beat is
the reference point is (almost) always the cycle

averageaht 9:30 PM
hmm I got the impression a cycle was a measure
not sure how I got that idea from the docs, but that might be why it's more difficult for me to wrap my head around how to do this

efairbanks 9:32 PM
Yeah, cycles are more like units of time, and time is malleable in Tidal.

efairbanks 9:32 PM
Tempo is defined in cycles per second rather than BPM.

anny 9:32 PM
cps 1 == bpm 60

yaxu 9:33 PM
only if you put four events per cycle

efairbanks 9:33 PM
^ depending on if you want to think of a cycle as a beat.

yaxu 9:34 PM
cps 1 feels like 120 bpm to me

averageaht 9:35 PM
why doesn't this work?
d1 $ slowcat [s "pluck*4" # up "[0 3 5 7]", s "pluck*2 ~ ~" # up "9 11 0 0 "]

yaxu 9:35 PM
but if you put three events in a cycle it'll feel like a different bpm

efairbanks 9:36 PM
@averageaht works for me
But!
This "pluck*2 ~ ~"
Does not mean
"pluck pluck ~ ~"

averageaht 9:36 PM
hmm for me it's playing the 9 twice

efairbanks 9:36 PM
It means
"[pluck pluck] ~ ~"
That's why it's playing the 9 twice.
The up pattern hasn't switched to 0 yet.

yaxu 9:37 PM
yes the second pluck will start 1/6th of the way into each cycle

efairbanks 9:37 PM
d1 $ slowcat [s "pluck*4" # up "[0 3 5 7]", s "pluck pluck ~ ~" # up "9 11 0 0 "] works the way you'd expect.
That's actually something that's a bit unclear about patterns. I've messed up thinking they would work that way fairly recently.

yaxu 9:38 PM
you can do "pluck!2 ~ ~" for that

efairbanks 9:39 PM
btw @averageaht if you want to apply the same progression to multiple patterns, you can do stuff like this: d1 $ slow 2 $ stack [s "bass:3(3,8)", s "pluck(5,8,2)" # up "24"] |*| up (slow 4 "0 5 7 0")


averageaht
9:39 PM
I can see how angle brackets are useful if cycle=beat. unfortunately they seem to just "not work" for me. I get:
no synth or sample named 'nil' could be found.
instrument not found: nil

yaxu
9:39 PM
well if you just want to repeat once you can do "pluck ! ~ ~"

anny 9:39 PM
pluck ! has the same effect - number is optional. pluck !!! also
i got owned by yaxu's perfect speed

efairbanks 9:40 PM
That's cool as heck. Is there way to tell Tidal to continue the last note instead of cutting it off in the case of using cut or synthdefs as well? I heard something about _ but last time I tried it, it didn't work for me.

oh actually there is @



2017-07-02 ----------------------------------------------------------------------------------------------------

mauro 2:46 PM
  added this Smalltalk snippet
Tidal {
  classvar ghci, ghciOutput;

  *new {
    ^this.start();
  }

  *start {
    if ( currentEnvironment.at(\tidal).isNil ){
      ~tidal = this;
      ghci = Pipe.new("ghci -XOverloadedStrings", "w");
      ghciOutput = Routine {
        var line = ghci.getLine;
        while(
          { line.notNil },
          {
            line.postln;
            line = ghci.getLine;
          }
        );
        ghci.close;
      };
    };
  }

  *send {
    |message|

    ghci.write("%\n".format(message));
    ghci.flush;
  }

  *stop {
    ghci.close;
    ~tidal = nil;
  }

}

mauro 2:47 PM
with a little help of @munshkr, it's possible to talk to tidal from supercollider

Tidal.start;
Tidal.send(":module Sound.Tidal.Context");
Tidal.send("(cps, getNow) <- bpsUtils");
Tidal.send("(d1,t1) <- superDirtSetters getNow");
Tidal.send(":set prompt ".format("tidal> ".quote));
Tidal.send("d1 $ sound % # release 0.25".format("kick".quote));
Tidal.send("d1 silence");
Tidal.stop;


datamads 11:55 PM
Sorry for reposting this but it’s such a dope little trick that it deserves the spotlight: https://medium.com/potac/reusable-code-in-tidalcycles-59b7a4dba30a


2017-07-07 ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


bgold
8:38 PM
Ooh, hadn't realized that there's no reason custom effects can't also use parameters like n (or freq).  Note-following filter:

~dirt.addModule('lpf2', {|dirtEvent| dirtEvent.sendSynth("dirt_lpf2" ++ ~dirt.numChannels,
    [cutoff2: ~cutoff2, freq:~freq, resonance:~resonance, out: ~out])}, {~cutoff2.notNil});

SynthDef("dirt_lpf2"++~dirt.numChannels, {|out, cutoff2=0, resonance, freq|
    var sig;
    sig = In.ar(out, ~dirt.numChannels);
    sig = RLPF.ar(sig, freq*(1+cutoff2), resonance.linexp(0, 1, 1, 0.001));
    ReplaceOut.ar(out, sig);
}).add;

2017-07-17
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

danielmkarlsson 5:51 PM
Hey gang.
I'd like to choose a new value for a sine envelope each time that particular envelope has completed its full duration.This example chooses a new value for the sine envelope each cycle which is not what I want:
d1 $ s "bass1:4*8" # cut 1 # gain (slow (choose [1..16]) $ scale 0 1 $ sine)

That said, you can fake it for a finite amount of time with seqP

let randomList sn len = map (ceiling . (* sn) . timeToRand) [1..len]
let cycleSines sn len = seqP $ map (\(a,b,c) -> (fromIntegral a, fromIntegral b, slow (fromIntegral c) sine)) $ zip3 x0 x1 r
       where x0 = take (length r - 1) $ scanl (+) 0 r
             x1 = scanl1 (+) r
             r = randomList sn len

so `cycleSines 16 100` will give you a series of sines with a `slow` in the range 1..16, and repeat for 100 sine cycles, which in this case is about to cycle count 850 (because of the random lengths).

Note that due to the way "random" numbers work, this will give the same exact sequence each time you use the same `sn` and `len`.  If you want to mix things up you can change up the `[1..len]` in randomList

`let randomList' sn len k = map (ceiling . (* sn) . timeToRand . (+ k)) [1..len]` would give a different result for different `k`

* [further footnote, `sine` starts its cycle at its midpoint, 0.5, if you want the waves to "reset" their cycle at 0.0 I think you can just rotate it, replacing `sine` with `0.25 ~> sine`]

bgold 3:48 AM
I realized mine doesn't work right - the phases aren't necessarily going to line up to be continuous.

ok try this
yaxu
10:57
eval these

import Sound.Tidal.Time

import Sound.Tidal.Utils

import Data.Maybe

yaxu
10:57
then this


let randArcs :: Int -> Pattern [Arc]
    randArcs n = do rs <- mapM (\x -> (pure $ (toRational x)/(toRational n)) <~ rand) [0 .. (n-1)]
                    let rats = map toRational rs
                        total = sum rats
                        pairs = pairUp $ accum 0 $ map ((/total)) rats
                    return $ pairs -- seqP $ map (\(a,b) -> (a,b,"x")) pairs
                      where pairUp [] = []
                            pairUp xs = (0,head xs):(pairUp' xs)
                            --
                            pairUp' [] = []
                            pairUp' (a:[]) = []
                            pairUp' (a:b:[]) = [(a,1)]
                            pairUp' (a:b:xs) = (a,b):(pairUp' (b:xs))
                            --
                            accum _ [] = []
                            accum n (a:xs) = (n+a):(accum (n+a) xs)
    randStruct n = splitQueries $ Pattern f
      where f (s,e) = mapSnds' fromJust $ filter (\(_,x,_) -> isJust x) $ as
              where as = map (\(n, (s',e')) -> ((s' + sam s, e' + sam s),
                                               subArc (s,e) (s' + sam s, e' + sam s),
                                               n)
                             ) $ enumerate $ thd' $ head $ arc (randArcs n) (sam s, nextSam s)
    compressTo (s,e) p = compress (cyclePos s, e-(sam s)) p
    substruct' :: Pattern Int -> Pattern a -> Pattern a
    substruct' s p = Pattern $ \a -> concatMap (\(a', _, i) -> arc (compressTo a' (inside (1/toRational(length (arc s (sam (fst a), nextSam (fst a))))) (rotR (toRational i)) p)) a') (arc s a)


then this

```d1 $ slow 16 $ substruct' (randStruct 16) $ sound "arpy*16" # gain (scale 0.5 1 sine) # speed (discretise 1 $ scale 1 2 rand)```



yaxu
12:05 AM
a bit better: d1 $ substruct (slow 8 $ randStruct 4) $ sound "arpy*8" # gain (scale 0.5 1 sine) # up (discretise 1 $ choose [0,7,12])
12:08
hmm d1 $ substruct (randStruct 2) $ sound "arpy*8" # gain (scale 0.5 1 sine) # up (discretise 1 $ choose [0,7,12,2])


yaxu 1:12 PM
The first pattern sounds like it has no metronomic pulse when you play it on its own but when you add the second pattern you can hear that the cp always aligns perfectly with the start of every fourth repetition of the first pattern

d1 $ slow 4 $ substruct' (randStruct 4) $ stack [n (off 0.125 (+7) $ "e2 d4 e3 [~ g3]") # sound "superzow" # pan saw # lpf (scale 500 1000 sine) # lpq 0.3]

d2 $ slow 4 $ sound "cp" # shape 0.6


danielmkarlsson
1:19 PM
That's cool.
1:20
I was messing around with this and it feels like it might be slightly off at times choosing values while not at zero. Try this:

d1
$ slow 16
$ substruct' (randStruct 4)
$ s "bass1*64"
# gain (scale 0 1 $ sine)
# up (discretise 1 $ choose [0,2,3,5,7,8,11] + choose [0,12,24,36])
# n (discretise 1 $ choose [5,7,9,13,19])
# pan (discretise 1 $ scale 0 1 $ rand)

Ah! bgolds solution with `0.25 ~> sine` sorted it!



