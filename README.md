# Gathering all the FEC bluk detailed data with the command line and regular expressions

[The Federal Election Commission contains a massive archive](http://www.fec.gov/finance/disclosure/ftpdet.shtml) of U.S. campaign finance data. The data is stored as flat, delmited text files. 

The FEC has many pages about data; the one in question is titled:[Detailed Files About Candidates, Parties and Other Committees](http://www.fec.gov/finance/disclosure/ftpdet.shtml)

Among the most interesting datasets are the individual contributions to campaigns -- the FEC has this from 1980 to the current election cycle. This dataset, which includes the identities and self-proclaimed occupations and residences of individual donors, contains a lot of potential insights about the kind of people who donate to particular campaigns. If you've ever used OpenSecrets, [this is where you can see the raw data](http://www.opensecrets.org/indivs/index.php).


## The problem

However, the data is separated by cycle, with each file stored as a separate zip file on the FEC's FTP server. When the data file is unpacked, the text is delimited with __pipes__, and the files are headerless, which means we have to attach the [data headers to each file ourselves](http://www.fec.gov/finance/disclosure/metadata/DataDictionaryContributionsbyIndividuals.shtml).

And that's before we've even inserted it into a database, nevermind done any actual analysis.

But if we could efficiently collect all the data, starting from 1980 to 2016, then we would have dozens of millions of records to analyze. Stories, such as this [LA Times feature into what Hollywood is donating in this cycle](http://graphics.latimes.com/2016-election-entertainment-donors/), could be repeated across every election cycle.

## The command-line solution

All your database skills won't matter if you can't even get the data into the database. There's plenty of ways to approach this programmatically, but in this writeup, I show how to do it via the Bash command-line, old programs like __seq__, and regular expressions, with a few third-party tools to make things a little more convenient:

- [csvkit](https://csvkit.readthedocs.org/en/0.9.1/) - the invaluable text-as-data toolkit, which we can use to analyze the raw text or simplify the database loading process. Its __csvgrep__ command let's us perform PCRE regex matches on individual columns (as opposed to the entire line).
- [ack](http://beyondgrep.com/) - a better version of __grep__, as it allows for the full suite of Perl-compatible regexes, along with the ability to make use of capture groups.
- [curl](https://curl.haxx.se/) - this is probably already installed on your system.
- uchardet - a [handy encoding-detection utility](https://github.com/BYVoid/uchardet)
- sqlite3 - again, another [ubiquitous piece of software, and all the database you need for this FEC data](https://www.sqlite.org/). 


When you've run all the steps, which make take 10 to 30 minutes depending on how fast your computer is, you'll have 3.7+GB in raw text, plus whatever database you create.


# Downloading the files



## How to get all the individual donor zip file names

Check out the page at [Detailed Files About Candidates, Parties and Other Committees](http://www.fec.gov/finance/disclosure/ftpdet.shtml).

Right-click copy any of the "Contributions by Individuals" files, e.g. `indiv16.zip` and copy the URL.


Each URL looks like this

ftp://ftp.fec.gov/FEC/2016/indiv16.zip


The pattern is simple, if `YYZZ` is a stand in for a year, such as `2016`:


    ftp://ftp.fec.gov/FEC/YYZZ/indivZZ.zip


## Generate a sequence of years

The [archive goes back to 1980](http://www.fec.gov/finance/disclosure/ftpdet.shtml#archive_link). We can use the __seq__ command for this:

~~~sh
$ seq 1980 2 2016
~~~

Output:
    
    1980
    1982
    1984
    1986
    1988
    1990
    1992
    1994
    1996
    1998
    2000
    2002
    2004
    2006
    2008
    2010
    2012
    2014
    2016    


## Use regex captured groups to create the pattern

The __ack__ tool has a useful `--output` flag that lets us use the power of regex captured groups:

~~~sh
$ seq 1980 2 2016 |
     ack '(\d\d(\d\d))' --output 'ftp://ftp.fec.gov/FEC/$1/indiv$2.zip'
~~~

The output:

~~~sh
ftp://ftp.fec.gov/FEC/1980/indiv80.zip
ftp://ftp.fec.gov/FEC/1982/indiv82.zip
ftp://ftp.fec.gov/FEC/1984/indiv84.zip
ftp://ftp.fec.gov/FEC/1986/indiv86.zip
ftp://ftp.fec.gov/FEC/1988/indiv88.zip
ftp://ftp.fec.gov/FEC/1990/indiv90.zip
ftp://ftp.fec.gov/FEC/1992/indiv92.zip
ftp://ftp.fec.gov/FEC/1994/indiv94.zip
ftp://ftp.fec.gov/FEC/1996/indiv96.zip
ftp://ftp.fec.gov/FEC/1998/indiv98.zip
ftp://ftp.fec.gov/FEC/2000/indiv00.zip
ftp://ftp.fec.gov/FEC/2002/indiv02.zip
ftp://ftp.fec.gov/FEC/2004/indiv04.zip
ftp://ftp.fec.gov/FEC/2006/indiv06.zip
ftp://ftp.fec.gov/FEC/2008/indiv08.zip
ftp://ftp.fec.gov/FEC/2010/indiv10.zip
ftp://ftp.fec.gov/FEC/2012/indiv12.zip
ftp://ftp.fec.gov/FEC/2014/indiv14.zip
ftp://ftp.fec.gov/FEC/2016/indiv16.zip
~~~

## Curl those FTP urls


~~~sh
$ seq 1980 2 2016 |
     ack '(\d\d(\d\d))' --output 'curl -O ftp://ftp.fec.gov/FEC/$1/indiv$2.zip'
~~~

All we're doing is adding the word "curl" that turns a plain old URL into something that can be executed:

The output:



~~~sh
curl -O ftp://ftp.fec.gov/FEC/1980/indiv80.zip
curl -O ftp://ftp.fec.gov/FEC/1982/indiv82.zip
curl -O ftp://ftp.fec.gov/FEC/1984/indiv84.zip
curl -O ftp://ftp.fec.gov/FEC/1986/indiv86.zip
curl -O ftp://ftp.fec.gov/FEC/1988/indiv88.zip
curl -O ftp://ftp.fec.gov/FEC/1990/indiv90.zip
curl -O ftp://ftp.fec.gov/FEC/1992/indiv92.zip
curl -O ftp://ftp.fec.gov/FEC/1994/indiv94.zip
curl -O ftp://ftp.fec.gov/FEC/1996/indiv96.zip
curl -O ftp://ftp.fec.gov/FEC/1998/indiv98.zip
curl -O ftp://ftp.fec.gov/FEC/2000/indiv00.zip
curl -O ftp://ftp.fec.gov/FEC/2002/indiv02.zip
curl -O ftp://ftp.fec.gov/FEC/2004/indiv04.zip
curl -O ftp://ftp.fec.gov/FEC/2006/indiv06.zip
curl -O ftp://ftp.fec.gov/FEC/2008/indiv08.zip
curl -O ftp://ftp.fec.gov/FEC/2010/indiv10.zip
curl -O ftp://ftp.fec.gov/FEC/2012/indiv12.zip
curl -O ftp://ftp.fec.gov/FEC/2014/indiv14.zip
curl -O ftp://ftp.fec.gov/FEC/2016/indiv16.zip
~~~


## Alternative bashing

Note: this walkthrough was written for relative Unix newbies who may not know all the intricacies of Unix-land or __curl__ subtleties. In later sections, I describe concatenating all the commands into a giant string delimited with semicolons...**that is not ideal**...but it's kind of commonsensical if all you know is regex. 

Here are more best practices approaches:


#### Make a bash script and bash it

1. Output the sequence of `curl` commands to a bash script file
2. `bash` that file

~~~sh
$ seq 1980 2 2016 |
     ack '(\d\d(\d\d))' \
        --output 'curl -O ftp://ftp.fec.gov/FEC/$1/indiv$2.zip' \
     > fecbashemall.sh

$ bash fecbashemall.sh
~~~

#### Use xargs


Forget printing everything out as a separate `curl` command; just use `ack` to pipe out a list of filenames into `xargs` and `curl`:

~~~sh
$ seq 1980 2 2016 |
     ack '(\d\d(\d\d))' \
        --output 'ftp://ftp.fec.gov/FEC/$1/indiv$2.zip' \
     | xargs curl -sO

~~~




## Concatenate into a semicolon delimited string

~~~sh
$ seq 1980 2 2016 |
     ack '(\d\d(\d\d))' \
         --output 'curl -O ftp://ftp.fec.gov/FEC/$1/indiv$2.zip;' |
     tr '\n' ' '
~~~

The messy output:

    curl -O ftp://ftp.fec.gov/FEC/1980/indiv80.zip; curl -O ftp://ftp.fec.gov/FEC/1982/indiv82.zip; curl -O ftp://ftp.fec.gov/FEC/1984/indiv84.zip; curl -O ftp://ftp.fec.gov/FEC/1986/indiv86.zip; curl -O ftp://ftp.fec.gov/FEC/1988/indiv88.zip; curl -O ftp://ftp.fec.gov/FEC/1990/indiv90.zip; curl -O ftp://ftp.fec.gov/FEC/1992/indiv92.zip; curl -O ftp://ftp.fec.gov/FEC/1994/indiv94.zip; curl -O ftp://ftp.fec.gov/FEC/1996/indiv96.zip; curl -O ftp://ftp.fec.gov/FEC/1998/indiv98.zip; curl -O ftp://ftp.fec.gov/FEC/2000/indiv00.zip; curl -O ftp://ftp.fec.gov/FEC/2002/indiv02.zip; curl -O ftp://ftp.fec.gov/FEC/2004/indiv04.zip; curl -O ftp://ftp.fec.gov/FEC/2006/indiv06.zip; curl -O ftp://ftp.fec.gov/FEC/2008/indiv08.zip; curl -O ftp://ftp.fec.gov/FEC/2010/indiv10.zip; curl -O ftp://ftp.fec.gov/FEC/2012/indiv12.zip; curl -O ftp://ftp.fec.gov/FEC/2014/indiv14.zip; curl -O ftp://ftp.fec.gov/FEC/2016/indiv16.zip; 



Which you can then just copy and paste. Then wait.



## Prep the combined data file

I'll assume you've saved them to a folder named `data/zips` relative to your current path. We want to save all the data to a file named: `data/all-individuals.txt`.

None of the individual individual contribution data files have headers. The headers are available at the following URL in CSV format:

[http://www.fec.gov/finance/disclosure/metadata/indiv_header_file.csv](http://www.fec.gov/finance/disclosure/metadata/indiv_header_file.csv)

Use __curl__ just to check it out:

~~~sh
$ curl -s http://www.fec.gov/finance/disclosure/metadata/indiv_header_file.csv
~~~

The output:

    CMTE_ID,AMNDT_IND,RPT_TP,TRANSACTION_PGI,IMAGE_NUM,TRANSACTION_TP,ENTITY_TP,NAME,CITY,STATE,ZIP_CODE,EMPLOYER,OCCUPATION,TRANSACTION_DT,TRANSACTION_AMT,OTHER_ID,TRAN_ID,FILE_NUM,MEMO_CD,MEMO_TEXT,SUB_ID



While all of the individual data files are __pipe delimited__, the `indiv_header_file.csv` is delimited with commas. Nothing that __sed__ or even just __tr__ can't fix. 

Let's convert the CSV headers into pipes, then redirect those pipe-delimited-headers into the new file, `data/all-individuals.txt`:


~~~sh
$ curl http://www.fec.gov/finance/disclosure/metadata/indiv_header_file.csv \
    | tr ',' '|' > data/all-individuals.txt
~~~



## List those zipped data files

By now, all of the zipped archives should have landed. Using curl's `-O` flag saves a given URL using the URL's basename. In other words, this:

    ftp://ftp.fec.gov/FEC/1980/indiv80.zip

is saved as:

    indiv80.zip


You could've run that batch `curl` sequence anywhere. I'll assume you did it in a subfolder named `data/zips`. If you jump up to the root directory, we can get a list of files using `ls` (or use __find__ if you're paranoid about how the filenames ended up):

~~~sh
$ ls data/zips/*.zip 
~~~


Output:

    data/zips/indiv00.zip data/zips/indiv10.zip data/zips/indiv82.zip data/zips/indiv92.zip
    data/zips/indiv02.zip data/zips/indiv12.zip data/zips/indiv84.zip data/zips/indiv94.zip
    data/zips/indiv04.zip data/zips/indiv14.zip data/zips/indiv86.zip data/zips/indiv96.zip
    data/zips/indiv06.zip data/zips/indiv16.zip data/zips/indiv88.zip data/zips/indiv98.zip
    data/zips/indiv08.zip data/zips/indiv80.zip data/zips/indiv90.zip


## Unzip those zipped data files

The __unzip__ command allows us to unzip contents directly to standard out with `-c`. The `q` (i.e. "quiet") option is also needed so that __unzip's__ progress messages aren't sent to stdout.


Note: As evidence of how bad my own Unix skills are, I didn't realize that it is quite trivial to unzip multiple things at once, [if you have a good understanding of how shell expansion works](https://chrisjean.com/unzip-multiple-files-from-linux-command-line/):

~~~sh
$ unzip -qc '*.zip'
~~~

You can ignore the instructions below unless you really like thinking about regex.



#### Clunky way of doing it with `ack`

Use __ack__ again to insert each filename into the proper command pattern:


~~~sh
$ ls data/zips/*.zip | 
  ack '(.+)' --output 'unzip -qc $1 >> data/all-individuals.txt'
~~~


The output:

    unzip -qc data/zips/indiv00.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv02.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv04.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv06.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv08.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv10.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv12.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv14.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv16.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv80.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv82.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv84.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv86.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv88.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv90.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv92.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv94.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv96.zip >> data/all-individuals.txt
    unzip -qc data/zips/indiv98.zip >> data/all-individuals.txt


Let's concat all those commands with double ampersand so that they run in a chain. The __tr__ command won't work since we're replacing newlines with a double ampersand (__tr__ only lets you substitute one-for-one characters). Sed won't work either because it doesn't work across lines. So just combine the two:

- `tr` to convert newlines to semicolons
- `sed` to convert semicolons to `&&` (the ampersands have to be escaped)
- `sed` one more time to remove the trailing ` && `

We could probably do it with __ack__ but best to stick to the bare tools when possible.

While we're here, let's throw in an "echo" for each file name so that we can get a sort of progress report.


~~~sh
$ ls data/zips/*.zip | 
    ack '(.+)' --output 'echo "$1" && unzip -qc $1 >> data/all-individuals.txt &&' \
    | tr '\n' ' '  \
    | sed 's/....$//g'
~~~

The output:

    echo "data/zips/indiv00.zip" && unzip -qc data/zips/indiv00.zip >> data/all-individuals.txt && echo "data/zips/indiv02.zip" && unzip -qc data/zips/indiv02.zip >> data/all-individuals.txt && echo "data/zips/indiv04.zip" && unzip -qc data/zips/indiv04.zip >> data/all-individuals.txt && echo "data/zips/indiv06.zip" && unzip -qc data/zips/indiv06.zip >> data/all-individuals.txt && echo "data/zips/indiv08.zip" && unzip -qc data/zips/indiv08.zip >> data/all-individuals.txt && echo "data/zips/indiv10.zip" && unzip -qc data/zips/indiv10.zip >> data/all-individuals.txt && echo "data/zips/indiv12.zip" && unzip -qc data/zips/indiv12.zip >> data/all-individuals.txt && echo "data/zips/indiv14.zip" && unzip -qc data/zips/indiv14.zip >> data/all-individuals.txt && echo "data/zips/indiv16.zip" && unzip -qc data/zips/indiv16.zip >> data/all-individuals.txt && echo "data/zips/indiv80.zip" && unzip -qc data/zips/indiv80.zip >> data/all-individuals.txt && echo "data/zips/indiv82.zip" && unzip -qc data/zips/indiv82.zip >> data/all-individuals.txt && echo "data/zips/indiv84.zip" && unzip -qc data/zips/indiv84.zip >> data/all-individuals.txt && echo "data/zips/indiv86.zip" && unzip -qc data/zips/indiv86.zip >> data/all-individuals.txt && echo "data/zips/indiv88.zip" && unzip -qc data/zips/indiv88.zip >> data/all-individuals.txt && echo "data/zips/indiv90.zip" && unzip -qc data/zips/indiv90.zip >> data/all-individuals.txt && echo "data/zips/indiv92.zip" && unzip -qc data/zips/indiv92.zip >> data/all-individuals.txt && echo "data/zips/indiv94.zip" && unzip -qc data/zips/indiv94.zip >> data/all-individuals.txt && echo "data/zips/indiv96.zip" && unzip -qc data/zips/indiv96.zip >> data/all-individuals.txt && echo "data/zips/indiv98.zip" && unzip -qc data/zips/indiv98.zip >> data/all-individuals.txt



Paste that puppy into your prompt. And when it's done, we can use good 'ol __wc__ to see how much data we collected:


~~~sh
$ wc data/all-individuals.txt
~~~

The output:

       25584847 124393123 3773595467 data/all-individuals.txt

124 million records, 3.7 gigabytes. Not bad for a few minutes of regular expressions and Bashery.


# Using csvkit


## Converting to comma-delimited format

If you don't like pipes, then you can use the __csvformat__ command to change the delimiter from `|` to clean, wholesome American commas.

However, because the FEC's text files aren't ASCII or UTF-8 encoded, we have to detect the encoding. I used Mozilla's __uchardet__:

~~~sh
$ uchardet data/all-individuals.txt
~~~

Output:

      windows-1252

Supply that into the `-e` flag of `csvformat`:

~~~sh
$  csvformat -e 'windows-1252' -d '|' \
        data/all-individuals.txt > data/all-individuals.csv
~~~


## Analyzing the data with csvkit

We can use `csvcut -n` to get an idea of what the columns are:

~~~sh
$ csvcut -n data/all-individuals.csv
~~~

The output shows the column names and their indexes:

~~~
    1: CMTE_ID
    2: AMNDT_IND
    3: RPT_TP
    4: TRANSACTION_PGI
    5: IMAGE_NUM
    6: TRANSACTION_TP
    7: ENTITY_TP
    8: NAME
    9: CITY
   10: STATE
   11: ZIP_CODE
   12: EMPLOYER
   13: OCCUPATION
   14: TRANSACTION_DT
   15: TRANSACTION_AMT
   16: OTHER_ID
   17: TRAN_ID
   18: FILE_NUM
   19: MEMO_CD
   20: MEMO_TEXT
   21: SUB_ID
~~~

You can see their official definitions from the FEC's data dictionary: [DataDictionaryContributionsbyIndividuals.shtml](http://www.fec.gov/finance/disclosure/metadata/DataDictionaryContributionsbyIndividuals.shtml)



### How many fucks given?

In 25+ years of campaign contributions, how many times have donors dropped the F-bomb? Finding out is as simple as this:

~~~sh
$ ack 'FUCK' data/all-individuals.csv
~~~


Or, maybe you're just wondering if the occupation and employer fields can be fully trusted; use `csvcut` to filter your search and limit the output:

~~~sh
$ csvcut -c EMPLOYER,OCCUPATION data/all-individuals.csv \
   | ack 'FUCK'
~~~

And here's the results, across 120+ million records:

        FUCK YOU,FUCK YOU
        FUCKFEDERALLAW,NONEOFYOURBUSINESS!
        FUCKFEDERALLAW,NONEOFYOURBUSINESS!
        SELF,FUCKING LOBBYIST
        NONE,OF YOUR FUCKING BUSINESS
        OF YOUR FUCKING BUSINESS,NONE
        SELF EMPLOYED MOTHER FUCKER,ELECTRICIAN
        HEARST,BAD-ASS FUCKING PROFESSIONAL CHILD
        SELF EMPLOYED MOTHER FUCKER,ELECTRICIAN



FWIW, it's worth searching across _all_ the fields for F-bomb fun -- as well as keeping the rest of the context.



### Finding Hollywood donations


It's not enough to restrict the `CITY` and `STATE` fields to New York and "Hollywood"; you'll want to search the `OCCUPATION` field -- though as we saw above, that field is self-described. 


For actors, directors, etc. who have donated at least in the 4-digits:

~~~sh
csvgrep -c 'OCCUPATION' -r '\bACTOR|ACTRESS|MOVIE|DIRECTOR|PRODUCER' \
  < data/all-individuals.csv \  
  | csvgrep -c TRANSACTION_AMT -r '\d{4}'
~~~

Warning: this takes a _long_ time, at least several minutes. The output includes nearly 190,000 rows.




## Inserting into SQLite


(Note: I'm not particularly skilled with SQLite from the command-line...after following the instructions on how to derive the schema using csvsql and create the table, you should probably just import the data using standard means, such as a GUI. csvsql may not be efficient enough in its implementation of row insertion to handle 124 million rows)

While doing data work with plaintext CSV is perfectly fine...within reason...maybe you feel safer with these hundreds of millions of records in SQL?

The awesome __csvsql__ command can be used to both quickly create the schema based on a CSV file, but also to do the actual bulk data inserts.

### Derive the schema

Grab first and last 2000 liens of the data file.

~~~sh
$ cat <(head -n 2000  data/all-individuals.csv) \
        <(tail -n 2000 data/all-individuals.csv) \
        < data/all-individuals.csv \
        | csvsql --tables individual_donors --no-constraints -i sqlite
~~~

Output:

    CREATE TABLE practitioners (
      "CMTE_ID" VARCHAR, 
      "AMNDT_IND" VARCHAR, 
      "RPT_TP" VARCHAR, 
      "TRANSACTION_PGI" VARCHAR, 
      "IMAGE_NUM" BIGINT, 
      "TRANSACTION_TP" VARCHAR, 
      "ENTITY_TP" VARCHAR, 
      "NAME" VARCHAR, 
      "CITY" VARCHAR, 
      "STATE" VARCHAR, 
      "ZIP_CODE" VARCHAR, 
      "EMPLOYER" VARCHAR, 
      "OCCUPATION" VARCHAR, 
      "TRANSACTION_DT" VARCHAR, 
      "TRANSACTION_AMT" INTEGER, 
      "OTHER_ID" VARCHAR, 
      "TRAN_ID" VARCHAR, 
      "FILE_NUM" VARCHAR, 
      "MEMO_CD" VARCHAR, 
      "MEMO_TEXT" VARCHAR, 
      "SUB_ID" BIGINT
    );

### Create the table with the schema

Run the same command, but redirect it into stdin of the __sqlite3__ command:

~~~sh
$ sqlite3 fec_data.sqlite \
    < <(cat <(head -n 2000  data/all-individuals.csv) \
        <(tail -n 2000 data/all-individuals.csv) \
        < data/all-individuals.csv \
        | csvsql --tables individual_donors --no-constraints -i sqlite)
~~~

### Insert the data in bulk

Note: Actually, probably not a good idea to try to insert all the data at once. Will try to figure out a more memory-safe way. The example below has been alterted to just insert the first 500,000 records:

~~~sh
$ cat data/all-individuals.csv \
   | head -n 500000 \
   | csvsql --no-constraints --no-inference  --no-create \
         --insert --tables individual_donors \
         --db sqlite:///fec_data.sqlite 
~~~

TODO: Just insert each individual archive file.


### Create indexes


~~~sh
sqlite3 fec_data.sqlite <<EOF
CREATE INDEX "indivdonors_cmte_id" 
    ON "individual_donors"("CMTE_ID");
CREATE INDEX "indivdonors_employer" 
    ON "individual_donors"("EMPLOYER");
CREATE INDEX "indivdonors_occupation" 
    ON "individual_donors"("OCCUPATION");
CREATE INDEX "indivdonors_transactiondate" 
    ON "individual_donors"("TRANSACTION_DT");

EOF
~~~


(more to come)
