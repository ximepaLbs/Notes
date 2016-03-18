#awk, sed, xargs examples

###copying PE32 files to different directory

file * | grep PE32 | awk '{i=split($1,a,":"); print a[1]}' | xargs cp -t dev-team-samples
