
# Proof of concept for migrating from docbook to rst


## Migration script

    for i in `ls` ; do FILE=`echo $i | cut -d '.' -f 1` ; /home/blkperl/.cabal/bin/pandoc -f docbook -t rst $i > ubuntu-serverguide/$FILE.rst ; done
