# Osysharp.TimeZone

One answer to "what day is it here?" for your whole app. Staff pick the time zone the team's days start in, and the day
a week starts, from a list on one settings page. Everything that counts in days reads it: reports start "today", "this
week" and "this month" at midnight there, and the daily digest goes out in the team's morning, in summer time as in
winter.

![The time zone, picked from the app's own list](docs/clock-picker.png)

```osy
use Osysharp.TimeZone@0;

policy ManagesClock => IsLead;                                         // required unless your app defines IsAdmin (its default)
app.Clock = new ClockSetup { TimeZone = TimeZoneInfo.FindSystemTimeZoneById("Europe/Stockholm") };  // optional: the clock until staff pick one

var clock = TheClock();
var today = clock.DayOf(DateTime.UtcNow);          // the date on the team's wall
var since = clock.StartOf(clock.WeekOf(today));    // this week began here, at midnight
```

| | |
|---|---|
| **A typed time zone** | a `Zone`, picked from your app's declared zones, never typed. A name that is not a time zone cannot be saved |
| **One settings page** | Settings › Time and date: the zone and the first day of the week, saved with who changed it, drawn with no page code |
| **Every kit on one clock** | `Osysharp.Reports` counts its periods and buckets on it, `Osysharp.Notifications` sends its digest by it |
| **Days that follow summer time** | `DayOf`, `StartOf` and `WeekOf` are the date, the midnight and the week start on the team's wall |
| **Your zones** | the platform's set by default; declare `zones { … }` in your app to offer exactly the ones your teams work in |
| **Read by everybody** | the clock is not a secret, so a public page can say "open until 17:00" on it; only `ManagesClock` changes it |

![On a phone, in the dark](docs/clock-settings-phone-dark.png)

`use Osysharp.TimeZone@0;` needs `Osysharp.Ui`.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.TimeZone;
using ClockKitTests;

[Test(Bakery)]
void before_anybody_sets_it_the_clock_is_the_apps_own() {
  runas (Mia) {
    var clock = TheClock();
    Assert.Equal(TimeZoneInfo.FindSystemTimeZoneById("Europe/Stockholm"), clock.TimeZone);
    Assert.Equal(DayOfWeek.Monday, clock.WeekStart);
  }
}
```

From [`tests/clock.test.osy`](tests/clock.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.TimeZone;
using ClockKitTests;

[Test(Bakery)]
void a_day_starts_at_midnight_on_the_teams_wall_and_follows_summer_time() {
  runas (Mia) {
    var clock = TheClock();
    // 23:30 UTC on 28 March is already 00:30 on the 29th in Stockholm — the night summer time begins.
    Assert.Equal(new DateOnly(2026, 3, 29), clock.DayOf(new DateTime(2026, 3, 28, 23, 30, 0)));
    // Midnight in winter is 23:00 UTC the day before; in summer, 22:00.
    Assert.Equal(new DateTime(2026, 1, 14, 23, 0, 0), clock.StartOf(new DateOnly(2026, 1, 15)));
    Assert.Equal(new DateTime(2026, 7, 14, 22, 0, 0), clock.StartOf(new DateOnly(2026, 7, 15)));
    // A week starts on Monday: Thursday 16 July 2026 is in the week of Monday 13 July.
    Assert.Equal(new DateOnly(2026, 7, 13), clock.WeekOf(new DateOnly(2026, 7, 16)));
  }
}
```

From [`tests/clock.test.osy`](tests/clock.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.TimeZone;
using ClockKitTests;

[Test(Bakery)]
void the_owner_moves_the_clock_and_every_reader_follows() {
  runas (Olle) {
    var row = new ClockSettings { TimeZone = TimeZoneInfo.FindSystemTimeZoneById("America/New_York"), WeekStart = DayOfWeek.Sunday };
    UnitOfWork.Commit();
  }
  runas (Mia) {
    var clock = TheClock();
    Assert.Equal(TimeZoneInfo.FindSystemTimeZoneById("America/New_York"), clock.TimeZone);
    // 03:00 UTC on 15 July is still the evening of the 14th in New York…
    Assert.Equal(new DateOnly(2026, 7, 14), clock.DayOf(new DateTime(2026, 7, 15, 3, 0, 0)));
    // …and a week that starts on Sunday puts Thursday 16 July in the week of Sunday 12 July.
    Assert.Equal(new DateOnly(2026, 7, 12), clock.WeekOf(new DateOnly(2026, 7, 16)));
  }
}
```

From [`tests/clock.test.osy`](tests/clock.test.osy).

## Learn more

- [The Osysharp.TimeZone reference](https://osysharp.com/reference/ui/clock-kit/)
- [Osysharp.TimeZone in the package catalogue](https://osysharp.com/packages/#osysharp-timezone)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
