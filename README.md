# Processing of visual and non-visual naturalistic spatial information in the "parahippocampal place area"

This repository contains the raw data and all code to generate the results in Häusler C.O. & Hanke M. (2021).

Kommentar: Adressen sind mitunter nicht öffentlich zugänglich; alle Befehle sind im Augenblick so, dass sie zum Ersten mal mit datalad aufgerufen werden, um das dataset von Grund auf aufzubauen

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



