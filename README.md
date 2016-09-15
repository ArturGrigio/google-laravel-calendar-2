# Manage events on a Google Calendar

This package is originally by [Spatie](https://spatie.be/opensource). It was using [Google API 1.1.7](https://github.com/google/google-api-php-client/tree/1.1.7), but I couldn't get the key to work properly, so I ended up reimplementing the `$client` with `OAuth2`.
Here is an example of what you can do with the repo:

```php
use Arturgrigio\GoogleCalendar\Event;

//create a new event
$event = new Event;

$event->name = 'A new event';
$event->startDateTime = Carbon\Carbon::now();
$event->endDateTime = Carbon\Carbon::now()->addHour();

$event->save();

// get all future events on a calendar
$events = Event::get(); 

$firstEvent = $events->first();
$firstEvent->name = 'updated name';
$firstEvent->save();

// create a new event
Event::create([
   'name' => 'A new event'
   'startDateTime' => Carbon\Carbon::now(),
   'endDateTime' => Carbon\Carbon::now()->addHour(),
]);

// delete an event
$event->delete();
```

## Installation

You can install the package via composer:

```bash
composer require arturgrigio/google-laravel-calendar-2
```

Next up the service provider must be registered:

```php
'providers' => [
    ...
    Arturgrigio\GoogleCalendar\GoogleCalendarServiceProvider::class,
];
```

Optionally the  `Spatie\GoogleCalendar\GoogleCalendarFacade` must be registered:

```php
'aliases' => [
	...
    'GoogleCalendar' => Arturgrigio\GoogleCalendar\GoogleCalendarFacade::class,
    ...
]
```

You must publish the configuration with this command:

```bash
php artisan vendor:publish --provider="Arturgrigio\GoogleCalendar\GoogleCalendarServiceProvider"
```

This will publish file called `laravel-google-calendar.php` in your config-directory with this contents:
```
<?php

return [

    /**
     * Path to a json file containing the credentials of a Google Service account.
     */
    'client_secret_json' => storage_path('app/laravel-google-calendar/client_secret.json'),

    /**
     *  The id of the Google Calendar that will be used by default.
     */
    'calendar_id' => '',
    
];
```

You'll need a OAuth Client ID `client_secret_xxxx...xxx.json` from your [google dev console](https://console.developers.google.com/apis/credentials).

## Usage

### Getting events

You can fetch all events by simply calling `Event::get();` this will return all events of the coming year. An event comes in the form of a `Spatie\GoogleCalendar\Event` object.

The fill signature of the function is:

```php
/**
 * @param \Carbon\Carbon|null $startDateTime
 * @param \Carbon\Carbon|null $endDateTime
 * @param array $queryParameters
 * @param string|null $calendarId
 *
 * @return \Illuminate\Support\Collection
 */
public static function get(Carbon $startDateTime = null, Carbon $endDateTime = null, array $queryParameters = [], string $calendarId = null) : Collection
```

The parameters you can pass in `$querParameters` are listed [on the documentation on `list` at the Google Calendar API docs](https://developers.google.com/google-apps/calendar/v3/reference/events/list#request).

### Creating an event

You can just new up a `Arturgrigio\GoogleCalendar\Event`-object

```php
$event = new Event;

$event->name = 'A new event';
$event->startDateTime = Carbon\Carbon::now();
$event->endDateTime = Carbon\Carbon::now()->addHour();

$event->save();
```

You can also call `create` statically:

```php
Event::create([
   'name' => 'A new event',
   'startDateTime' => Carbon\Carbon::now(),
   'endDateTime' => Carbon\Carbon::now()->addHour(),
]);
```

This will create an event with a specific start and end time. If you want to create a full day event you must use `startDate` and `endDate` instead of `startDateTime` and `endDateTime`.

```php
$event = new Event;

$event->name = 'A new full day event';
$event->startDate = Carbon\Carbon::now();
$event->endDate = Carbon\Carbon::now()->addDay();

$event->save();
```

### Getting a single event

Google assigns a unique id to every single event. You can get this id by getting events using the `get` method and getting the `id` property on a `Arturgrigio\GoogleCalendar\Event`-object:
```php
// get the id of the first upcoming event in the calendar.
$calendarId = Event::get()->first()->id;
```

You can use this id to fetch a single event from Google:
```php
Event::find($calendarId)
```

### Updating an event

Easy, just change some properties and call `save()`:

```php
$event = Event::find($eventId)

$event->name = 'My updated title' 
$event->save();
```

### Deleting an event

Nothing to it!

```php
$event = Event::find($eventId)

$event->delete()
```

### Limitations

The Google Calendar API provides many options. This package doesn't support all of them. For instance recurring events cannot be managed properly with this package. If you stick to creating events with a name and a date you should be fine.

## Testing

``` bash
$ composer test
```

## Credits

- [Freek Van der Herten](https://github.com/freekmurze)
- [All Contributors](../../contributors)
- [Spatie](https://spatie.be/opensource)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
