# INEWS - ISPF News and Notification
*INEWS* is an ISPF News based dialog.  It presents news items to the
ISPF user upon request, or optionally can be integrated into ISR@PRIM
so that is presents new news items overtly. It can also be enabled, by
the user, to perform a check once each day when ISPF starts.

## Installation:

  - execute the $RECEIVE exec to expand the EXEC, MSGS, PANELS, and
    SKELS members into partitioned datasets
  - edit/tailor the INEWS exec for your shop
    - dataset names
    - admin userids
    - more
  - the print option uses an exec called %PPRINT which is not distributed
    with this package as it is unique to each environment. Find *print*
    in the INEWS exec for the routine to change for your environment.
  - copy the EXEC, MSGS, PANELS and SKELS members into production level
    libraries
  - Allocate the two news libraries for the default news
  - Allocate the two news libraries for the test news
  - experiment using tso %inews test admin and tso %inews test
    - this works with the test libraries until you are comfortable
      and are ready to move to the default (production) libraries
  - run the ADMIN dialog to generate some news items
    - tso %inews admin

The user will have an option when the news is displayed, via the CHECK
command, to enable automatic, once per day, checking for new and unread
news items. This is helpful if the optional ISPF panel updates are not
implemented and gives individual users control over the automatic check.

See the optional section at the end for information n how to force the
automatic checking for new news items once each day at ispf startup.

## Administration

To administer this application enter %INEWS ADMIN to bring up the
administrative dialog.  Create new news items and then SAVE.  You may
DELETE or UPDATE an existing news item. Note that if you use UPDATE be
aware the end user may not see the update as the tracking of which
news items the user has seen is stored in the users ISPF Profile in a
table.

The administrative options are documented on the table display panel.

Note: Use option C (clone) to make a copy of an existing news item to
      make updates to it. This gives the item a new number and thus it
      will appear with a status of New for users that automatically
      check for New and Unread items at ISPF startup. The old item
      remains or can be deleted using option D.

Detour: The dialog maintains a small ISPF table with one row in the
users ISPF Profile dataset.  This table contains a flag for each news
item indicating the users status in relation to the item. Thus if a news
item has been marked read then the user will not see any UPDATES to that
news item unless they explicitly view it.

Note that the ADMIN updates are made to a working copy of the table
dataset and at SAVE time all updates are copied into production.

## User Interface

The user interface is a simple ISPF table.  If the INEWS command is
entered without operands then the entire table is displayed.  If the
INEWS command is entered with the NEW keyword then only NEW and UNREAD
items will be displayed. This is what the %inewsck exec does.

The following command and line options are available:

```
Command Options:  Find xxx    Search xxx   Refresh  XAll (mark all ignored)
Line Options:     S -Select   U -Unread   X -Ignore   P -Print
```

Find   will find the specified character string in the displayed title
       text.

Refresh will rebuild the news table and is designed for use after
        a Search

Search will look for the specified word/string in all members and display
       only those members which have that word.  This search is done
       using ISPF SuperC (search-for).

XALL   will mark all new and unread News items as Ignored.

S      Select the current news item to browse
U      Mark the current news item unread
X      Mark the current news item ignored
P      Print the current news item

## Usage:

INEWS is entered by executing the Rexx procedure INEWS and providing one
or more of the valid parameters:

Syntax:   INEWS parameter1 parameter2

Parameter1:
              NEWS     (default) defines the NEWS table to use
              TEST     use the TEST news table

Optional Parameter2:
              NEW       Display the news ISPF table only if NEW or
                        UNREAD items are in the table for the  user
              DEBUG     Used to turn on REXX tracing for debugging purposes
              ADMIN     Enable the administration application
              TEST      Uses a test news table for testing purposes.
              FORCE     If the table is locked by ADMIN and the lock is
                        obsolete (for some reason that left the member
                        LOCK in the news data set) this option will
                        override the lock for administration purposes.

## Optional
The INEWS exec does require an update to ISR@PRIM and ISP@MSTR to
force a call to the %inewck exec to check for new and unread news items.

Add the following statements to the ISR@PRIM and ISP@MSTR ISPF Panels in
the )INIT section between the two existing statements:

```
&ZSCLMPRJ = &Z      /* TUTORIAL INDEX - 1ST PAGE     @L1A*/  <== Existing

IF (&ZSCREEN = 1)                /* lbd */
  IF (&ZNEWDATE = &ZDATE)        /* lbd */
     IF (&Z$SAVE ^= &Z)          /* lbd */
         &ZCMD   = &Z$SAVE       /* lbd */
         &Z$SAVE = &Z            /* lbd */
         .RESP     = ENTER       /* lbd */
  IF (&ZNEWDATE NE &ZDATE)       /* lbd */
      &Z$SAVE   = &ZCMD          /* lbd */
      &ZCMD     = 'NF'           /* lbd */
      &ZPDFINIT = 'DONE'         /* lbd */
      .RESP     = ENTER          /* lbd */
      &ZNEWDATE = &ZDATE         /* lbd */
      VPUT (ZNEWDATE) PROFILE    /* lbd */

IF (&ZLOGO = 'YES')                              /* CK@MJC*/  <== Existing
```

Then in the )PROC section add this statement in the list of statements
for menu options:

```
        NF,'CMD(%INEWSZST) NEWAPPL(ISR)'
```

What this will do is the following:

1. Each time the panel is called a check will be made for ZSCREEN 1 and
   variable ZNEWSPAN and if it is DONE then that processing will be
   skipped.
2. If not DONE then setup the option to be NF to invoke the %INEWSCK
   exec to check for new and unread news items.
