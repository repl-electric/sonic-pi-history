require 'json'
ROOM_SOURCE = "/Users/josephwilk/Workspace/josephwilk/ruby/bright/data.json"
ROOM        = JSON.parse(File.read(ROOM_SOURCE))
CAMERA_1    = ROOM.map{|event| event["people"]["cam_1"]}
CAMERA_2    = ROOM.map{|event| event["people"]["cam_2"]}
CAMERA_3    = ROOM.map{|event| event["people"]["cam_3"]}
SOUND_MAX   = ROOM.map{|event| event["sound"]["max"]}
SOUND_AVG   = ROOM.map{|event| event["sound"]["avg"]}
SOUND_MIN   = ROOM.map{|event| event["sound"]["min"]}
PEOPLE_MIN = 0
PEOPLE_MAX = (CAMERA_1+CAMERA_2+CAMERA_3).max
camera1_rate = (CAMERA_1).map{|p| linear_map_fn(PEOPLE_MIN, PEOPLE_MAX, 0.0, 1.0).(p)}
camera2_rate = (CAMERA_2).map{|p| linear_map_fn(PEOPLE_MIN, PEOPLE_MAX, 0.0, 1.0).(p)}
camera3_rate = (CAMERA_3).map{|p| linear_map_fn(PEOPLE_MIN, PEOPLE_MAX, 0.0, 1.0).(p)}
@timeline = (ring *[camera1_rate, camera2_rate, camera3_rate].transpose)
@sound_time = (ring *SOUND_AVG.map{|p| linear_map_fn(0, SOUND_AVG.max, 0.0, 1.0).(p)})
@cam1_smoothed ||= nil
@cam2_smoothed ||= nil
@cam3_smoothed ||= nil
@avg_smoothed  ||= nil
bar = 1.0
ELECTRIC_SAMPLES = (ring :elec_triangle, :elec_lo_snare, :elec_hi_snare, :elec_mid_snare, :elec_cymbal, :elec_soft_kick, :elec_filt_snare, :elec_fuzz_tom, :elec_chime, :elec_bong, :elec_twang, :elec_wood, :elec_blip, :elec_blip2, :elec_ping, :elec_bell, :elec_flip, :elec_tick, :elec_hollow_kick, :elec_twip, :elec_plip, :elec_blup)
SNARE_SAMPLES =    (ring :sn_zome, :sn_dolf, :sn_dub, :elec_snare)
PICK_SCALE = ["Fs", "B", "E", "A", "D", "G", "C", "F", "Bb", "Eb", "Ab", "Db", "Gb"].choose
MODE = ["minor_pentatonic", "major_pentatonic"].choose
BASS_SYNTHS = [:dsaw, :prophet, :saw] 
DRUMS_STATE = true
live_loop :metronome do
  cue :whole;cue :half;cue :quarter
  sleep bar/4.0
  cue :quarter
  sleep bar/4.0; cue :half
  cue :quarter
  sleep bar/4.0
  cue :quarter
  sleep bar/4.0
end
live_loop :camera1 do |idx|
  DRUMS_LIFE = {kick: :elec_soft_kick, perc_samples: ELECTRIC_SAMPLES[0..8], beat_stretching: (ring 0.8, 0.8, 0.8, -1.0)}
  score = @timeline.hook(:inc_time)[0]
  @cam1_smoothed ||= score
  @cam1_smoothed = smoothed_score(score, @cam1_smoothed)
  DRUMS_STATE = (@cam1_smoothed > 0.1) ? true : false 
  case @cam1_smoothed
  when 0.8..1.0
    DRUMS_LIFE[:perc_samples] = ELECTRIC_SAMPLES[8..16]
    DRUMS_LIFE[:beat_stretching] = (ring 1.0, 0.5, 0.25, -1.0)
    DRUM_DENSITY = 8
  when 0.5..0.8
    DRUMS_LIFE[:perc_samples] = ELECTRIC_SAMPLES.shuffle[0..8]
    DRUMS_LIFE[:beat_stretching] = (ring 0.8, 0.8, 0.8, 1.0)
    DRUM_DENSITY = 4
  when 0.2..0.5
    DRUMS_LIFE[:perc_samples]    = ELECTRIC_SAMPLES[0..8].reverse
    DRUMS_LIFE[:beat_stretching] = (ring 0.2, 0.1, 0.1, -1.0)
    DRUM_DENSITY = 4
  else
    DRUMS_LIFE[:perc_samples] = ELECTRIC_SAMPLES[0..8]
    DRUMS_LIFE[:beat_stretching] = (ring 0.2, 0.1, 0.2, 0.3)
    DRUM_DENSITY = 4
  end
  with_fx :level, amp: @cam1_smoothed*0.9 do
    sync :whole
    @timeline.tick(:inc_time)
    if DRUMS_STATE
      sample DRUMS_LIFE[:kick], rate: beat_stretch(DRUMS_LIFE[:kick],(ring 1.0, 0.9, 0.99, 0.9).tick(:beat)), amp: @cam1_smoothed*4.0, start: 0.0
      with_fx(:echo, decay: 8){with_fx(:bitcrusher, sample_rate: 20000){sample((knit :drum_cymbal_soft,1,nil,15).tick(:s), amp: 0.1)}} if @cam1_smoothed >= 0.8
    end
    s1, s2, s3 = *DRUMS_LIFE[:perc_samples].tick(:beat)
    density(DRUM_DENSITY) do |n|
      with_fx :lpf, cutoff: 80, mix: 1-@cam1_smoothed do
        with_fx (knit :none, 7, :echo, 1).tick(:beat), decay: bar*2 do
          with_fx :bitcrusher, bits: (ring *(8..10).to_a).tick(:bits) do
            if DRUMS_STATE
              perc_pat = (knit s1, 3,nil,1, s2, 2, nil,1, s3, 2)
              sample perc_pat.tick(:beat), amp: (n==1) ? 0.55 : 0.35, pan: (ring 0.25,-0.25).tick(:pan), rate: beat_stretch(perc_pat.hook(:beat),0.1)
            end
            sleep bar
            if DRUMS_STATE
              sample (knit SNARE_SAMPLES[0], 4).tick(:snare_sample), amp: @cam1_smoothed*0.2, rate: beat_stretch(SNARE_SAMPLES[0], (knit 1.0, 2, 0.5, 2).tick(:snare))
            end
          end
        end
      end
    end
   idx+=1
  end
end
live_loop :camera2 do |cam2_idx|
  score = @timeline.hook(:inc_time)[0]
  @cam2_smoothed ||= score
  @cam2_smoothed = smoothed_score(score, @cam2_smoothed)
  BASS_SYNTHS = BASS_SYNTHS.shuffle if (cam2_idx % 120 == 0)
  case @cam2_smoothed
  when  0.6..1.0
    BASS_SYNTHS = [:dsaw, :prophet] 
    @bass_line = (chords_for(find_scale_root(3), MODE).shuffle+[chord(find_scale_root(3),'sus2'),chord(find_scale_root(3),'sus4'), chord(find_scale_root(3),'M')])
  when 0.0..0.15
    BASS_SYNTHS = [:fm, :beep] 
    @bass_line = (chords_for(find_scale_root(1), MODE)).shuffle
  else
    BASS_SYNTHS = [:dsaw, :prophet, :saw] 
    @bass_line = (chords_for(find_scale_root(1), MODE)+chords_for(find_scale_root(2), MODE).shuffle+[chord(find_scale_root(2),'sus2'),chord(find_scale_root(2),'sus4'),chord(find_scale_root(2),'M')])
  end
  with_fx :level, amp: @cam2_smoothed+0.1 do
    @bass_line.tick(:bass)
    repeat_times = (ring 4,4,4,2,4,2)
    (repeat_times.tick(:time)).times do
      sync :whole
      @timeline.tick(:inc_time)
    end
    with_fx(:reverb, room: 1.0, mix: 0.9, room_lag: 1.0) do |fx_reverb|
      with_synth BASS_SYNTHS[0] do
        play @bass_line.ring.hook(:bass)[0..2], attack: 0.01, amp: 0.8, release: bar*4, res: 0.5, cutoff: 60
        if repeat_times.hook(:time) == 2
          sample (ring :perc_snap, :perc_snap2).tick(:perc), amp: 0.01
          control fx_reverb , room: 0.0, mix: 0.8
          sleep bar/4.0
          sample (ring :perc_snap, :perc_snap2).tick(:perc), amp: 0.01

          control fx_reverb , room: 0.1, mix: 0.9
          sleep bar/4.0
          sample (ring :perc_snap, :perc_snap2).tick(:perc), amp: 0.01

          control fx_reverb , room: 0.2, mix: 0.8
          sleep bar/4.0
          sample (ring :perc_snap, :perc_snap2).tick(:perc), amp: 0.02

          control fx_reverb , room: 0.5, mix: 0.7
          sleep bar/4.0
          control fx_reverb , room: 1.0, mix: 0.9
        else
          with_transpose(12) do
            with_fx :slicer, mix: @cam2_smoothed do
              with_fx :pitch_shift do
                with_synth BASS_SYNTHS[1] do
                  play @bass_line.ring.hook(:bass)[-1], attack: 0.01, amp: 0.5, release: bar*0.3
                end
              end
            end
            4.times{
              sync :half
              control fx_reverb, damp: (rrand 0.0, 1.0)
            }
          end
        end
      end
    end
  end
  cam2_idx+=1
end
live_loop :camera3 do
  score = @timeline.hook(:inc_time)[2]
  @cam3_smoothed ||= score
  @cam3_smoothed = smoothed_score(score, @cam3_smoothed)
  with_fx :level, amp: @cam3_smoothed+0.5 do
    APEG = {}
    case @cam3_smoothed
    when 0.0..0.3
      APEG = {release: 0.1, attack: 0.01,  synth: :hollow, amp_factor: 7.0 }
    when 0.3..0.7
      APEG = {release: 0.25, attack: 0.25, synth: :hollow, amp_factor: 10.0}
    when 0.7..1.0
      APEG = {release: 2.0, attack: 0.01,  synth: :beep, amp_factor: 0.8}
    end
    1.times{ sync (knit :half, 2, :quarter, 2).tick(:time) }
    @timeline.tick(:inc_time)
    with_fx :pitch_shift, pitch_dis: 0.0001, time_dis: 0.01, window_size: 1.5  do
      with_fx (knit :echo, 2, :reverb, 2).tick(:room), decay: @cam3_smoothed*10, pre_amp: 0.4 do
        with_synth(APEG[:synth]) do
          note =  stretch_it(scale(find_scale_root(1), MODE).shuffle, (ring 6,2).tick(:dups1),
                             scale(find_scale_root(1), MODE).shuffle, (ring 2,6).tick(:dups2),

                             scale(find_scale_root(2), MODE).shuffle, (ring 4,4).tick(:dups1),
                             scale(find_scale_root(2), MODE).shuffle, (ring 2,6).tick(:dups1),

                             scale(find_scale_root(3), MODE).shuffle, (ring 4,4).tick(:dups1),
                             scale(find_scale_root(3), MODE).shuffle, (ring 2,6).tick(:dups1)).tick(:apeg)
          play note, release: APEG[:release], amp: APEG[:amp_factor], attack: APEG[:attack]
        end
      end
    end
  end
end
live_loop :room_texture do
  score = @sound_time.hook(:time)
  @avg_smoothed ||= score
  @avg_smoothed = smoothed_score(score, @avg_smoothed)
  octave = @avg_smoothed >= 0.9 ? 3 : 2
  with_fx :level, amp: @avg_smoothed*0.25 do
    with_fx :reverb, room: 1.0 do
      with_fx :pitch_shift, pitch_dis: 0.01, time_dis: 0.1, window_size: 0.5 do
        with_fx :slicer, phase: (ring bar*1, bar*4,bar*8).tick(:slice), smooth: 1.0 do
          with_synth(:dark_ambience) do
            play chords_for(find_scale_root(octave), MODE).tick(:slice), amp: (@avg_smoothed*13.0), attack: bar*4.0, release: bar*4.0, sustain: bar
          end
          with_synth(:pnoise) do
            play chords_for(find_scale_root(0), MODE).tick(:slice), amp: 0.55, attack: bar*4.0, release: bar*2.0, sustain: bar/2.0
          end
        end
        8.times {sleep bar; @sound_time.tick(:time)}
      end
    end
  end
end