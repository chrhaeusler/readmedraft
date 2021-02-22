# Processing of visual and non-visual naturalistic spatial information in the "parahippocampal place area"

This repository contains the raw data and all code to generate the results in Häusler C.O. & Hanke M. (2021).

Kommentar: 
1) alle Befehle sind im Augenblick so, dass sie zum Ersten mal mit datalad aufgerufen werden, um das dataset von Grund auf aufzubauen
2) Adressen sind mitunter nicht öffentlich zugänglich


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
    # segment the location annotation using timings of the audio-visual movie
    datalad run \
    -i inputs/studyforrest-data-annotations/researchcut/locations.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    avmovie avmovie \
    '{outputs}'
    
    # segment the location annotation using timings of the audio-description
    datalad run \
    -i inputs/studyforrest-data-annotations/researchcut/locations.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    aomovie aomovie \
    '{outputs}'
    
    # install annotation of speech as subdataset
    datalad install -d . -s juseless.inm7.de:/data/group/psyinf/studyforrest-speechannotation inputs/studyforrest-speechannotation
    # download the annotation as BIDS-conform .tsv
    datalad get inputs/studyforrest-speechannotation/annotation/fg_rscut_ad_ger_speech_tagged.tsv
    
    # segment the speech annotation using timings of the audio-description
    # NOW HOSTED ON OSF.io
    datalad run \
    -i inputs/studyforrest-speechannotation/annotation/fg_rscut_ad_ger_speech_tagged.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    aomovie aomovie \
    '{outputs}'
    
    # segment the speech annotation using timings of the audio-visual movie
    # NOW hosted on osf.io
    datalad run \
    -i inputs/studyforrest-speechannotation/annotation/fg_rscut_ad_ger_speech_tagged.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    avmovie avmovie \
    '{outputs}'
    
## manual addition of confound annotations and a script
    # add low-level confound files of audio-visual movie manually & save (folder "avconfounds")
    datalad save -m 'add low-level confound files for audio-visual movie to /events/segments'
    # add low-level confound files of audio-description manually & save (folder "aoconfounds")
    datalad save -m 'add low-level confound files for audio-description to /events/segments'
    # add script that code/confounds2onsets.py
    datalad save -m 'add script that converts & copies confound files to onsets directories'

## copy confound annotations into onsets directories of movie and audio-description
    # consider directory for corresponding fMRI run
    # rename filenames according to naming conventions of regressors
    datalad run \
    -i events/segments \
    -o events/onsets \
    ./code/confounds2onsets.py -i '{inputs}' -o '{outputs}'

