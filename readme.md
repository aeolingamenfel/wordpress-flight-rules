# Flight Rules for WordPress Development

A set of scenario-specific instructions for developing WordPress themes and
plugins.

#### What are "flight rules"?

A [guide for astronauts](http://www.jsc.nasa.gov/news/columbia/fr_generic.pdf) (now, programmers using WordPress) about what to do when things go wrong.

>  *Flight Rules* are the hard-earned body of knowledge recorded in manuals that list, step-by-step, what to do if X occurs, and why. Essentially, they are extremely detailed, scenario-specific standard operating procedures. [...]

> NASA has been capturing our missteps, disasters and solutions since the early 1960s, when Mercury-era ground teams first started gathering "lessons learned" into a compendium that now lists thousands of problematic situations, from engine failure to busted hatch handles to computer glitches, and their solutions.

&mdash; Chris Hadfield, *An Astronaut's Guide to Life*.

*Inspired by [this Git flight rules doc](https://github.com/k88hudson/git-flight-rules)*

---

**Table of Contents**

  - [Database Manipulation](#database-manipulation)
    - [I want to create a new table in the database](#i-want-to-create-a-new-table-in-the-database)

## Database Manipulation

### I want to create a new table in the database

Let's say that you need to persistently store something for your theme or
plugin, so you want to add a table to the WordPress SQL database.

The first step is to register a function on the WordPress activation hook
(which is called when your theme/plugin is activated). This function should
set up the table that you want to store data in.

First, setup the activation hook:

```PHP
register_activation_hook(__FILE__, 'example_install_db');
```

Then setup the `example_install_db` function (modifying the name to suit your
project, of course). Below is an example showing a basic table, but the main
query string uses standard SQL syntax, so you can expand on it as necessary:

```PHP
function example_install_db() {
    // we need to have this in order to reference the $wpdb helper
    global $wpdb;

    // this is whatever you want the table to be named
    $table_name = $wpdb->prefix . "example_table";

    // get the global character set for WordPress
    $charset_collate = $wpdb->get_charset_collate();

    // Careful editing this string, WordPress is EXTREMELY picky about
    // formatting, down to individual space counts.
    // See: https://codex.wordpress.org/Creating_Tables_with_Plugins for more
    // information.
    $sql = "CREATE TABLE $table_name (
        id mediumint(9) NOT NULL AUTO_INCREMENT,
        time datetime DEFAULT '0000-00-00 00:00:00' NOT NULL,
        PRIMARY KEY  (id)
    ) $charset_collate;";

    // this source file provides the dbDelta() function we're about to use
    require_once(ABSPATH . 'wp-admin/includes/upgrade.php');

    // dbDelta() will only change the database if necessary
    dbDelta($sql);
}
```
