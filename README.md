# Processing of visual and non-visual naturalistic spatial information in the "parahippocampal place area": from raw data to results

This repository contains the raw data and all code to generate the results in Häusler C.O. & Hanke M. (2021).

Kommentar: 
1) alle Befehle sind im Augenblick so, dass sie zum Ersten mal mit datalad aufgerufen werden, um das dataset von Grund auf aufzubauen (s. "datalad save" of manually added scripts; alle befehle in "datalad run" gewrappt)
2) die Reihenfolge ist minimal gemäß des roten Fadens in der Method Section im Paper angepasst worden (d.h. readme weicht minimal von der git history im Analysis-Dataset ab)
3) Adressen sind mitunter nicht öffentlich zugänglich


## clone the DataLad dataset
comment: probably the correct start of the readme; where do we wanna host the dataset?
following are the steps to create the dataset from scratch

## install subdatasets and get the raw data

    # install subdataset that provides motion corrected fMRI data from the audio-visual movie and its audio-description
    datalad install -d . -s https://github.com/psychoinformatics-de/studyforrest-data-aligned inputs/studyforrest-data-aligned
    # download 4D fMRI data (and motion correction parameters of the movie) 
    datalad get inputs/studyforrest-data-aligned/sub-??/in_bold3Tp2/sub-??_task-a?movie_run-?_bold*.*
    
comment: following data source might not publicly accessible ?!
    
    # install subdataset that provides the original 7 Tesla data to get the motion correction parameters of the audio-descriptio
    datalad install -d . -s juseless.inm7.de:/data/project/studyforrest/collection/phase1 inputs/phase1
    datalad get inputs/phase1/sub???/BOLD/task001_run00?/bold_dico_moco.txt

    # install subdataset "template & transforms", and download the relevant images
    datalad install -d . -s https://github.com/psychoinformatics-de/studyforrest-data-templatetransforms inputs/studyforrest-data-templatetransforms
    datalad get inputs/studyforrest-data-templatetransforms/sub-*/bold3Tp2/
    datalad get inputs/studyforrest-data-templatetransforms/templates/*
    
    # install subdataset "studyforrest-data-annotations" that contains the annotation of cuts & locations as subdataset 
    # and "code/researchcut2segments.py" that we need to segment the (continuous) annotations
    datalad install -d . -s https://github.com/psychoinformatics-de/studyforrest-data-annotations inputs/studyforrest-data-annotations
    
comment: following source for the speech-anno is juseless not osf.io
    
    # install the annotation of speech as subdataset
    datalad install -d . -s juseless.inm7.de:/data/group/psyinf/studyforrest-speechannotation inputs/studyforrest-speechannotation
    # download the annotation as BIDS-conform .tsv
    datalad get inputs/studyforrest-speechannotation/annotation/fg_rscut_ad_ger_speech_tagged.tsv
    
## segmenting of continuous annotations
    # segment the location annotation using timings of the audio-visual movie segments
    datalad run \
    -i inputs/studyforrest-data-annotations/researchcut/locations.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    avmovie avmovie \
    '{outputs}'
    
    # segment the speech annotation using timings of the audio-description segments
    datalad run \
    -i inputs/studyforrest-speechannotation/annotation/fg_rscut_ad_ger_speech_tagged.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    aomovie aomovie \
    '{outputs}'
    
comment: following "av" as input-timing is 'correct' based on the new annotation created by montreal forced aligner because
random shift +/- 40 ms around 0 of whole movie vs. whole audio-description; previous manual annotation was shifted manually before performing the annotation because there was a lag of 40 ms at the beginning; but it turned out be +/- 40 around 40 over the course of the whole stimulus 
hence: the shifted timings in individual segments compared to the whole stimuli is far more importante that the jitter of whole movie vs. whole audio-description.

    # for control contrasts, segment the speech annotation using timings of the audio-visual movie segments
    datalad run \
    -i inputs/studyforrest-speechannotation/annotation/fg_rscut_ad_ger_speech_tagged.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    avmovie avmovie \
    '{outputs}'

comment: following "ao" as input-timing is 'correct' based on the new annotation created by montreal forced aligner

    # for control contrasts, segment the location annotation using timings of the audio-description segments
    datalad run \
    -i inputs/studyforrest-data-annotations/researchcut/locations.tsv \
    -o events/segments \
    ./inputs/studyforrest-data-annotations/code/researchcut2segments.py \
    '{inputs}' \
    aomovie aomovie \
    '{outputs}'
    
## manual addition of confound annotations and a script that gets the annotation in shape for the subsequent FEAT analyses
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
    
## copy event files to folders of individual subjects

    # manually add the script that creates directories & handles the copying
    datalad save -m 'add script that creates subject directories and copies FSL event files  into it'

    # create subjects folders & copy events with timing of the audio-visual movie
    datalad run \
    -m "create subject folders & copy event files to it" \
    ./code/onsets2subfolders.py \
    -fmri 'inputs/studyforrest-data-aligned/sub-??/in_bold3Tp2/sub-??_task-aomovie_run-1_bold.nii.gz' \
    -onsets 'events/onsets/avmovie/run-?/*.txt' \
    -o './'
    
    # copy events with timing of the audio-description 
    datalad run \
    -m "copy event files with audio-description with movie timings to subject folders" \
    ./code/onsets2subfolders.py \
    -fmri 'inputs/studyforrest-data-aligned/sub-??/in_bold3Tp2/sub-??_task-aomovie_run- 1_bold.nii.gz' \
    -onsets 'events/onsets/aomovie/run-?/*.txt' \
    -o './'
    
## manually add the design file templates
   
    # manually add the script that creates first level individual design files from template
    datalad save -m 'add python script that creates individual (1st level) design files from templates'

   
    # analyses in group space, level 1-3 (e.g. 1st-lvl_movie-ppa-grp.fsf, 2nd-lvl_movie-ppa-grp.fsf, 3rd-lvl_movie-ppa-grp-1.fsf)
    # both steps include adding the bash scripts that take 2nd level templates as input and create design-files in individual directories 
    # (e.g. generate_2nd-lvl-design_movie-ppa-grp.sh)
    datalad save -m 'add FSL design files (lvl 1-3) for movie (group)'
    datalad save -m 'add FSL design files (lvl 1-3) for audio (group)'

    # analyses in subject space, level 1-2 (e.g. 1st-lvl_movie-ppa-ind.fsf, 2nd-lvl_movie-ppa-ind.fsf)
    # both steps include adding the bash scripts that take 2nd level templates as input and create design-files in individual directories 
    # (e.g. generate_2nd-lvl-design_movie-ppa-ind.sh)
    datalad save -m 'add FSL design files (lvl 1-2) for movie (individuals)'
    datalad save -m 'add FSL design files (lvl 1-2) for audio (individuals)'
    
## from templates, create design files for individual subjects
  
    # movie, group space, first level
    datalad run \
    -m 'for movie analysis (group), create individual (1st level) design files from template' \
    code/generate_1st-lvl-design.py \
    -fmri 'inputs/studyforrest-data-aligned/sub-01/in_bold3Tp2/sub-01_task-avmovie_run-1_bold.nii.gz' \
    -design 'code/1st-lvl_movie-ppa-grp.fsf'

    # movie, group space, second level
    datalad run \
    -m "for movie analysis (group), generate individual 2nd lvl design files from template" \
    "./code/generate_2nd-lvl-design_movie-ppa-grp.sh"

    # audio-description, group space, first level
    datalad run \
    -m 'for audio analysis (group), create individual 1st level design files from template' \
    code/generate_1st-lvl-design.py \
    -fmri 'inputs/studyforrest-data-aligned/sub-01/in_bold3Tp2/sub-01_task-aomovie_run-1_bold.nii.gz' \
    -design 'code/1st-lvl_audio-ppa-grp.fsf'

    # audio-description, group space, second level
    datalad run \
    -m "for audio analysis (group), generate individual 2nd lvl design files from template" \
    "./code/generate_2nd-lvl-design_audio-ppa-grp.sh"

    # movie, subject space, first level
    datalad run \
    -m 'for movie analysis (individuals), create individual 1st level design files from template' \
    code/generate_1st-lvl-design.py \
    -fmri 'inputs/studyforrest-data-aligned/sub-01/in_bold3Tp2/sub-01_task-avmovie_run-1_bold.nii.gz' \
    -design 'code/1st-lvl_movie-ppa-ind.fsf'

    # movie, subject space, second level
    datalad run \
    -m "for movie analysis (individuals), generate individual 2nd lvl design files from template" \
    "./code/generate_2nd-lvl-design_movie-ppa-ind.sh"

    # audio-description, subject space, first level
    datalad run \
    -m 'for audio analysis (individuals), create individual 1st level design files from template' \
    code/generate_1st-lvl-design.py \
    -fmri 'inputs/studyforrest-data-aligned/sub-01/in_bold3Tp2/sub-01_task-aomovie_run-1_bold.nii.gz' \
    -design 'code/1st-lvl_audio-ppa-ind.fsf'

    # audio-description, subject space, second level
    datalad run \
    -m "for audio analysis (individuals), generate individual 2nd lvl design files from template" \
    "./code/generate_2nd-lvl-design_audio-ppa-ind.sh"
    
## manually add bash script that handles custom templates for FEAT
    datalad save -m "add script that add templates & transformation matrices to 1st lvl result directories of Feat"
    
## running the analyses via condor_submit on a computer cluster & manually save results
    # add file "condor-commands-for-cm.txt" that contains the following commands to manually submit the subsequent analyses to HTCondor
    datalad save -m "add txt file with instructions for manually starting Condor Jobs from CM"
    
    # movie, group space, first level
    condor_submit code/compute_1st-lvl_movie-ppa-grp.submit
    # in .feat-directories, create templates and transforms
    ./code/reg2std4feat inputs/studyforrest-data-templatetransforms bold3Tp2 grpbold3Tp2 sub-*/run-?_movie-ppa-grp.feat
    # movie, group space, second level
    condor_submit code/compute_2nd-lvl_movie-ppa-grp.submit
    # movie, group space, third level
    condor_submit code/compute_3rd-lvl_movie-ppa-grp.submit
    # save results of first to third level
    datalad save -m '3rd lvl results movie (group)'
    
    # audio-description, group space, first level
    condor_submit code/compute_1st-lvl_audio-ppa-grp.submit
    # in .feat-directories, create templates and transforms
    ./code/reg2std4feat inputs/studyforrest-data-templatetransforms bold3Tp2 grpbold3Tp2 sub-*/run-?_audio-ppa-grp.feat
    # audio-description, group space, second level
    condor_submit code/compute_2nd-lvl_audio-ppa-grp.submit    
    # audio-description, group space, third level
    condor_submit code/compute_3rd-lvl_audio-ppa-grp.submit
    # save results of first to third level
    datalad save -m '3rd lvl results audio (group)'

    # movie, subject space, first level
    condor_submit code/compute_1st-lvl_movie-ppa-ind.submit
    # in .feat-directories, create templates and transforms
    ./code/reg2std4feat inputs/studyforrest-data-templatetransforms bold3Tp2 bold3Tp2 sub-*/run-?_movie-ppa-ind.feat
    # movie, subject space, second level
    condor_submit code/compute_2nd-lvl_audio-ppa-ind.submit
    # save results of first to second level
    datalad save -m '2nd lvl results audio (individuals)'
    
    # audio-description, subjects space, first level
    condor_submit code/compute_1st-lvl_audio-ppa-ind.submit
    # in .feat-directories, create templates and transforms
    ./code/reg2std4feat inputs/studyforrest-data-templatetransforms bold3Tp2 bold3Tp2 sub-*/run-?_audio-ppa-ind.feat
    # audio-description, subject space, second level
    condor_submit code/compute_2nd-lvl_audio-ppa-ind.submit
    # audio-description, group space, third level
    datalad save -m '2nd lvl results audio (individuals)'   
    # save results of first to third level
    
## comment: some cleaning that we did
    git annex unused
    git annex dropunused all --force
    datalad drop --nocheck sub*/*.feat/filtered_func_data.nii.gz
    datalad drop --nocheck sub*/*.feat/stats/res4d.nii.gz

## comment: all subdatasets in ./inputs are still installed and contain data
see /data/project/studyforrest_ppa/*











    
    
    


