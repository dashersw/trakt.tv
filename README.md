# trakt.tv
**Trakt.tv API wrapper for Node.js, featuring:**

- [All Trakt.tv API v2 methods](https://github.com/vankasteelj/trakt.tv/wiki/Supported-methods)
- Promises
- Forget JSON, use Objects, Arrays and Strings directly
- Enhanced protection against: CSRF (session riding) and XSS (content spoofing) attacks, using Crypto and Sanitizer
- [Plugin extension](https://github.com/vankasteelj/trakt.tv/wiki/Available-plugins)

*For more information about Trakt.tv API, read http://docs.trakt.apiary.io/*

## Example usage

#### Setup

    npm install trakt.tv

#### Initialize
```js
const Trakt = require('trakt.tv');

const trakt = new Trakt({
  client_id: <the_client_id>,
  client_secret: <the_client_secret>,
  redirect_uri: null,   // defaults to 'urn:ietf:wg:oauth:2.0:oob'
  api_url: null         // defaults to 'https://api.trakt.tv'
});
```

#### Generate Auth URL
```js
const traktAuthUrl = trakt.get_url();
```

#### Verify code (and optionally state) from returned auth
```js
trakt.exchange_code('code', 'csrf token (state)').then(result => {
    // contains tokens & session information
    // API can now be used with authorized requests
});
```

#### Alternate OAUTH "device" method
```js
trakt.get_codes().then(poll => {
    // poll.verification_url: url to visit in a browser
    // poll.user_code: the code the user needs to enter on trakt

    // verify if app was authorized
    return trakt.poll_access(poll);
}).catch(error => {
    // error.message == 'Expired' will be thrown if timeout is reached
});
```

#### Refresh token
```js
trakt.refresh_token().then(results => {
    // results are auto-injected in the main module cache
});
```

#### Storing token over sessions
```js
// get token, store it safely.
const token = trakt.export_token();

// injects back stored token on new session.
trakt.import_token(token).then(newTokens => {
    // Contains token, refreshed if needed (store it back)
});
```

#### Actual API requests
See methods in [methods.json](https://github.com/vankasteelj/trakt.tv/blob/master/methods.json) or [wiki](https://github.com/vankasteelj/trakt.tv/wiki/Supported-methods).

```js
trakt.calendars.all.shows({
    start_date: '2015-11-13',
    days: '7',
    extended: 'full'
}).then(shows => {
    // Contains Object{} response from API (show data)
});
```

```js
trakt.search.text({
    query: 'tron',
    type: 'movie,person'
}).then(response => {
    // Contains Array[] response from API (search data)
});
```

```js
trakt.search.id({
    id_type: 'imdb',
    id: 'tt0084827'
}).then(response => {
    // Contains Array[] response from API (imdb data)
});
```

#### Using pagination
You can extend your calls with `pagination: true` to get the extra pagination info from headers.

```js
trakt.movies.popular({
    pagination: true
}).then(movies => {
    /**
    movies = Object {
        data: [<actual data from API>],
        pagination: {
            item-count: "80349",
            limit: "10",
            page: "1",
            page-count: "8035"
        }
    }
    **/
});
```

Note: _this will contain `data` and `pagination` for all calls, even if no pagination is available. it's typically for really advanced use only_

#### Load plugins
When calling `new Trakt()`, include desired plugins in an object (must be installed from npm):

```js
const trakt = new Trakt({
    client_id: <the_client_id>,
    client_secret: <the_client_secret>,
    plugins: {  // load plugins
        images: require('trakt.tv-images')
    }
    options: {  // pass options to plugins
        images: {
            smallerImages: true
        }
    }
});
```

The plugin can be accessed with the key you specify. For example `trakt.images.get()`.

#### Write plugins
See the [wiki page](https://github.com/vankasteelj/trakt.tv/wiki/Write-plugins-for-trakt.tv).

#### Notes
- You can use 'me' as username if the user is authenticated.
- Timestamps (such as token _expires_ property) are Epoch in milliseconds.

## LICENSE

The MIT License (MIT) - author: Jean van Kasteel <vankasteelj@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.