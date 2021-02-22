# Processing of visual and non-visual naturalistic spatial information in the "parahippocampal place area"

This repository contains the raw data and all code to generate the results in Häusler C.O. & Hanke M. (2021).

Kommentar: 
1) alle Befehle sind im Augenblick so, dass sie zum Ersten mal mit datalad aufgerufen werden, um das dataset von Grund auf aufzubauen
2) die Reihenfolge ist minimal gemäß des roten Fadens in der Method Section im Paper angepasst worden (d.h. weicht minimal von der history im Analysis-Dataset ab)
3) Adressen sind mitunter nicht öffentlich zugänglich


## install subdatasets and get the raw data

    # install subdatasets providing fMRI data from the audio-visual movie and its audio-description
    datalad install -d . -s https://github.com/psychoinformatics-de/studyforrest-data-aligned  inputs/studyforrest-data-aligned
    # download 4D fMRI data and motion correction parameters
    datalad get inputs/studyforrest-data-aligned/sub-??/in_bold3Tp2/sub-??_task-a?movie_run-?_bold*.*
    
    # get motion correction parameter files for phase 1 data
    # THIS DATA MIGHT NOT BE PUBLIC ?!
    datalad install -d . -s juseless.inm7.de:/data/project/studyforrest/collection/phase1 inputs/phase1
    datalad get inputs/phase1/sub???/BOLD/task001_run00?/bold_dico_moco.txt

    # install & get templates and transforms
    datalad install -d . -s https://github.com/psychoinformatics-de/studyforrest-data-templatetransforms inputs/studyforrest-data-templatetransforms
    datalad get inputs/studyforrest-data-templatetransforms/sub-*/bold3Tp2/
    datalad get inputs/studyforrest-data-templatetransforms/templates/*
    
    # install annotation of cuts & locations (as subdataset in psychoinformatics-de/studyforrest-data-annotations)
    datalad install -d . -s https://github.com/psychoinformatics-de/studyforrest-data-annotations inputs/studyforrest-data-annotations
    
     # install annotation of speech as subdataset
    datalad install -d . -s juseless.inm7.de:/data/group/psyinf/studyforrest-speechannotation inputs/studyforrest-speechannotation
    # download the annotation as BIDS-conform .tsv
    datalad get inputs/studyforrest-speechannotation/annotation/fg_rscut_ad_ger_speech_tagged.tsv
    
# segmenting of continuous annotations
    # segment the location annotation using timings of the audio-visual movie
    datalad run \
    -i inputs/studyforrest-data-annotations/researchcut/locations.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    avmovie avmovie \
    '{outputs}'
    
    # segment the speech annotation using timings of the audio-description
    # NOW HOSTED ON OSF.io
    datalad run \
    -i inputs/studyforrest-speechannotation/annotation/fg_rscut_ad_ger_speech_tagged.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    aomovie aomovie \
    '{outputs}'
    
    # for control contrasts, segment the location annotation using timings of the audio-description
    datalad run \
    -i inputs/studyforrest-data-annotations/researchcut/locations.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    aomovie aomovie \
    '{outputs}'
    
    # for control contrasts, segment the speech annotation using timings of the audio-visual movie
    # NOW hosted on osf.io
    datalad run \
    -i inputs/studyforrest-speechannotation/annotation/fg_rscut_ad_ger_speech_tagged.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    avmovie avmovie \
    '{outputs}'
    
## manual addition of confound annotations and a script that gets the annotation in shape for the subsequent analyses
    # add low-level confound files of audio-visual movie manually & save (folder "avconfounds")
    datalad save -m 'add low-level confound files for audio-visual movie to /events/segments'
    # add low-level confound files of audio-description manually & save (folder "aoconfounds")
    datalad save -m 'add low-level confound files for audio-description to /events/segments'

## convert confound annotations into FSL onset files
    # add script code/confounds2onsets.py
    datalad save -m 'add script that converts & copies confound files to onsets directories'
    # perform the conversion considering the directories of corresponding fMRI runs and
    # rename according to conventions used in FSL-design files
    datalad run \
    -i events/segments \
    -o events/onsets \
    ./code/confounds2onsets.py -i '{inputs}' -o '{outputs}'
    
## create onsets files from the segmented annotation of cuts & locations
    # add the script that performs the conversion
    datalad save -m 'add script that creates event files for FSL from the segmented location annotation'

    # create event onset files from segmented location annotation (timings of audio-visual movie)
    datalad run \
    -m "create the event files with movie timing" \
    -i events/segments/avmovie \
    -o events/onsets \
    ./code/locationsanno2onsets.py \
    -ind '{inputs}' \
    -inp 'locations_run-?_events.tsv' \
    -outd '{outputs}'

    # create event onset files from segmented location annotation (timings of audio-description)
    datalad run \
    -m "create the event files with audio-track timing" \
    -i events/segments/aomovie \
    -o events/onsets \
    ./code/locationsanno2onsets.py \
    -ind '{inputs}' \
    -inp 'locations_run-?_events.tsv' \
    -outd '{outputs}'
   
## create onsets files from the segmented annotation of speech
    # add the script that performs the conversion
    datalad save -m 'add script that creates event files for FSL from the segmented speech annotation'

    # create event onset files from segmented speech annotation (timings of audio-visual movie)
    datalad run \
    -i events/segments/avmovie \
    -o events/onsets \
    ./code/speechanno2onsets.py \
    -ind '{inputs}' \
    -inp 'fg_rscut_ad_ger_speech_tagged_run-*.tsv' \
    -outd '{outputs}'

    # create event onset files from segmented speech annotation (timings of audio-description)
    datalad run \
    -i events/segments/aomovie \
    -o events/onsets \
    ./code/speechanno2onsets.py \
    -ind '{inputs}' \
    -inp 'fg_rscut_ad_ger_speech_tagged_run-*.tsv' \
    -outd '{outputs}'
