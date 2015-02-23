# percolate:synced-cron

A simple cron system for [Meteor](http://meteor.com). It supports syncronizing jobs between multiple processes. In other words, if you add a job that runs every hour and your deployment consists of multiple app servers, only one of the app servers will execute the job each time (whichever tries first).

## Installation

``` sh
$ meteor add percolate:synced-cron
```

## API

### Basics

To write a cron job, give it a unique name, a schedule an a function to run like below. SyncedCron uses the fantastic [later.js](http://bunkat.github.io/later/) library behind the scenes. A Later.js `parse` object is passed into the schedule call that gives you a huge amount of flexibility for scheduling your jobs, see the [documentation](http://bunkat.github.io/later/parsers.html#overview). 

``` js
SyncedCron.add({
  name: 'Crunch some important numbers for the marketing department',
  schedule: function(parser) {
    // parser is a later.parse object
    return parser.text('every 2 hours');
  }, 
  job: function() {
    var numbersCrunched = CrushSomeNumbers();
    return numbersCrunched;
  }
});
```

To start processing your jobs, somewhere in your project add:

``` js
SyncedCron.start();
```

If you want a job to run only once or a limited number of times, you can either remove it inside the job function or create a limited schedule:

```javascript
SyncedCron.add({
  name: 'myJob',
  schedule: function(parser) {
    // EITHER set a limited schedule
    var date = new Date;
    date.setHours(date.getHours() + 12);

    // (this Later.js object only contains a single event)
    return parser.recur().on(date).fullDate();
  }, 
  job: function() {
    // (do something)

    // ... OR remove the job inside function
    SyncedCron.remove('myJob');
  }
});
```


### Advanced

SyncedCron uses a collection called `cronHistory` to syncronize between processes. This also serves as a useful log of when jobs ran along with their output or error. A sample item looks like:

``` js
{ _id: 'wdYLPBZp5zzbwdfYj',
  intendedAt: Sun Apr 13 2014 17:34:00 GMT-0700 (MST),
  finishedAt: Sun Apr 13 2014 17:34:01 GMT-0700 (MST),
  name: 'Crunch some important numbers for the marketing department',
  startedAt: Sun Apr 13 2014 17:34:00 GMT-0700 (MST),
  result: '1982 numbers crunched'
}
```

Call `SyncedCron.nextScheduledAtDate(jobName)` to find the date that the job
referenced by `jobName` will run next.

Call `SyncedCron.remove(jobName)` to remove and stop running the job referenced by jobName.

Call `SyncedCron.stop()` to remove and stop all jobs.

### Configuration

Modify the object `SyncedCron.options` to set configuration entries. Defaults are: 

``` js
  SyncedCron.options = {
    //Log job run details to console
    log: true,

    //Name of collection to use for synchronisation and logging
    collectionName: 'cronHistory',

    //Default to using localTime
    utc: false, 

    //TTL in seconds for history records in collection to expire
    //NOTE: Unset to remove expiry but ensure you remove the index from 
    //mongo by hand
    //
    //ALSO: SyncedCron can't use the `_ensureIndex` command to modify 
    //the TTL index. The best way to modify the default value of 
    //`collectionTTL` is to remove the index by hand (in the mongo shell 
    //run `db.cronHistory.dropIndex({startedAt: 1})`) and re-run your 
    //project. SyncedCron will recreate the index with the updated TTL.
    collectionTTL: 172800
  }
```

## Caveats

Beware, SyncedCron probably won't work as expected on certain shared hosting providers that shutdown app instances when they aren't receiving requests (like Heroku's free dyno tier or Meteor free galaxy).

## Contributing

Write some code. Write some tests. To run the tests, do:

``` sh
$ meteor test-packages ./
```

## License 

MIT. (c) Percolate Studio

Synced Cron was developed as part of the [Verso](http://versoapp.com) project.
