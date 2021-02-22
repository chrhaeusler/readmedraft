# Processing of visual and non-visual naturalistic spatial information in the "parahippocampal place area"

This repository contains the raw data and all code to generate the results in Häusler C.O. & Hanke M. (2021).

## install subdatasets and get the raw data

    # install subdatasets providing fMRI data from the audio-visual movie and its audio-description
    datalad install -d . -s https://github.com/psychoinformatics-de/studyforrest-data-aligned  inputs/studyforrest-data-aligned

    # download 4D fMRI data and motion correction parameters
    datalad get inputs/studyforrest-data-aligned/sub-??/in_bold3Tp2/sub-??_task-a?movie_run-?_bold*.*



