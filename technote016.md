# Tech Note 016 - ion path
### IO Network file system and the path microlanguage

* 'with' or 'to' defined in standardized modules exporting io network filesystem objects
* equivalent to 9fs
 1 standard message format
 2 per-context namspace with bind and mount
 3 union mounts, etc.
 4 chainable


 () sub operation
 [] group parallel
 {} group sequential
 /**/ recursive walk until 
 * match any 
 < localized.operation on /some/data 
 $cmdpath provides list of assumed module prefixes when module part is unspecified

$cmdpath="bin:sbin:Files:Tools:OXP"

A dynamic module may update its cmd section to include launchers for binaries in a path
An interpreted module may update its cmd section to include newly defined procedures

 < uniq < sort < grep ERROR /var/log/*.log                    performs work locally
 /mnt/remote/*/var/log < uniq < sort < grep ERROR /*.log      performs work remotely
 /mnt/remote/*/repo/project < diff (/work/Test.Mod) (/old/Test.Mod)
 

 3  tf flush      - <>
 4  tf attach  @  i <>
 5  tf walk    /  i name 
 6  tf open       i <>
 7  tf create  ^  i name
 8  tf read    ?  i offset:amt
 9  tf write   !  i offset:data
 10 tf clunk   %  i <>
 11 tf remove  ~  i <>
 12 tf stat    #  i <>
 13 tf wstat   %  i data


$i=/foo^bar            create bar in foo and assign reference to i
$i!:"Hello world"       write "Hello world" to i at current offset
/foo/bar?              read contents of bar in foo
$i~                    remove i (bar) from foo
$j=/foo/baz            assign /foo/baz to reference j
$j?300:512             read 512 bytes from offset 300
/mnt/remote/*/etc/hosts!-1:(cat (/etc/hostip?:) " " (/etc/hostname?:))
                       adds an entry for current machine to everybody's hosts file


/build/**/*.o~           remove .o files in subdirectories of build  /*

operations on ids in messages map to operations on pointers to records


PROCEDURE to (VAR tag,fid: INTEGER; VAR op: ARRAY OF CHAR; param: IONOP): IONOP;

result string may be found in op


foo := to( <root>,1,"ls",NULL);


PROCEDURE do (VAR tag, fid: INTEGER; VAR op: ARRAY OF CHAR): DEES, DOSE;


