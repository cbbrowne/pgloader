# Loading Fixed Width Data File with pgloader

Some data providers still use a format where each column is specified with a
starting index position and a given length. Usually the columns are
blank-padded when the data is shorter than the full reserved range.

## The Command

To load data with [pgloader](http://pgloader.tapoueh.org/) you need to
define in a *command* the operations in some details. Here's our example for
loading Fixed Width Data, using a file provided by the US census.

You can find more files from them at the
[Census 2000 Gazetteer Files](http://www.census.gov/geo/maps-data/data/gazetteer2000.html).

Here's our command:

    LOAD ARCHIVE
       FROM http://www.census.gov/geo/maps-data/data/docs/gazetteer/places2k.zip
       INTO postgresql:///pgloader
    
       BEFORE LOAD DO
         $$ drop table if exists places; $$,
         $$ create table places
           (
              usps      char(2)  not null,
              fips      char(2)  not null,
              fips_code char(5),
              loc_name  varchar(64)
           );
         $$
    
       LOAD FIXED
            FROM FILENAME MATCHING ~/places2k.txt/
                 WITH ENCODING latin1
                 (
                 -- name   start  length
                    usps       0  2,
                    fips       2  2,
                    fips_code  4  5,
                    loc_name   9 64,
                    p         73  9,
                    h         82  9,
                    land      91 14,
                    water    105 14,
                    ldm      119 14,
                    wtm      131 14,
                    lat      143 10,
                    long     153 11
                 )
            INTO postgresql:///pgloader?places
                 (
    	        usps, fips, fips_code,
                    loc_name text using (right-trim loc_name)
                 );

You can see the full list of options in the
[pgloader reference manual](pgloader.1.html), with a complete description
of the options you see here.

## The Data

This command allows loading the following file content, where we are only
showing the first couple of lines:

    AL0100124Abbeville city                                                       2987     1353      40301945        120383   15.560669    0.046480 31.566367 -85.251300
    AL0100460Adamsville city                                                      4965     2042      50779330         14126   19.606010    0.005454 33.590411 -86.949166
    AL0100484Addison town                                                          723      339       9101325             0    3.514041    0.000000 34.200042 -87.177851
    AL0100676Akron town                                                            521      239       1436797             0    0.554750    0.000000 32.876425 -87.740978
    AL0100820Alabaster city                                                      22619     8594      53023800        141711   20.472605    0.054715 33.231162 -86.823829
    AL0100988Albertville city                                                    17247     7090      67212867        258738   25.951034    0.099899 34.265362 -86.211261
    AL0101132Alexander City city                                                 15008     6855     100534344        433413   38.816529    0.167342 32.933157 -85.936008

## Loading the data

Let's start the `pgloader` command with our `census-places.load` command file:

    $ pgloader census-places.load
    ... LOG Starting pgloader, log system is ready.
    ... LOG Parsing commands from file "/Users/dim/dev/pgloader/test/census-places.load"
    ... LOG Fetching 'http://www.census.gov/geo/maps-data/data/docs/gazetteer/places2k.zip'
    ... LOG Extracting files from archive '//private/var/folders/w7/9n8v8pw54t1gngfff0lj16040000gn/T/pgloader//places2k.zip'
    
           table name       read   imported     errors            time
    -----------------  ---------  ---------  ---------  --------------
             download          0          0          0          1.494s
              extract          0          0          0          1.013s
          before load          2          2          0          0.013s
    -----------------  ---------  ---------  ---------  --------------
               places      25375      25375          0          0.499s
    -----------------  ---------  ---------  ---------  --------------
    Total import time      25375      25375          0          3.019s

We can see that [http://pgloader.tapoueh.org](pgloader) did download the
file from its HTTP URL location then *unziped* it before the loading itself.

Note that the output of the command has been edited to facilitate its
browsing online.
