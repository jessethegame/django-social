Django Auths is a framework for social authentication within Django.

Several third party backends are included.

Installation
------------

      #settings.py

      # When more than one service is specified, multiple accounts can be connected
      # to a single composite user.
      AUTHS = {
        'facebook': {
          'app': 'social.facebook',
          'creds': {
            'key': 'YOURAPPKEY',
            'secret': 'YOURAPPSECRET',
        }
        'twitter': {
          'app': 'social.twitter',
          'creds': {
            'key': 'YOURAPPKEY',
            'secret': 'YOURAPPSECRET',
        }
        'googleplus': {
          'app': 'social.googleplus',
          'creds': {
            'key': 'YOURAPPKEY',
            'secret': 'YOURAPPSECRET',
        }
      }

      MIDDLEWARE_CLASSES += ('auths.middleware.AuthMiddleware',)

Protocols & Backends
--------------------
`protocols.Protocol`
provides an interface for (third party) authorization
sources. It specifies the authentication flow with two methods:

`request(request, callback_url)`
This returns a url which will initiate the authentication flow. Afterwards, the
user should be redirected to the callback_url.

`callback(request)`
Completes the authentication flow and returns a dictionary with the resulting
credentials.

`logout(request)`
Defaults to Djangos native logout mechanism but allows alternatives by
overriding this method.

`backends.Backend`
Provides an interface for turning credentials into a user
object of its associated service provider.

`authenticate(\*\*creds)`
Returns a User object authenticated by given credentials.

`get_user(self, pk)`
Returns an instance of the User model found in the module of the backend.

Configuration
-----
For a single source of authentication, add the desired backend and provider
credentials to settings.

    #settings.py
    AUTHENTICATION_BACKENDS += ('auths.backends.facebook.Backend',)

    PROVIDERS = {
      'facebook': {
        'key': 'YOURAPPKEY',
        'secret': 'YOURAPPSECRET',
      }
    }

Accounts
--------

Social auth provides accounts to enable composite user authentication. This
means that you can plug in as many providers as you like and have users connect
one user account for each of them to a single user.

    #settings.py
    AUTHENTICATION_BACKENDS += ('auths.backends.accounts.Backend',)

    PROVIDERS = {
      'facebook': {
        'creds': {
          'key': 'YOURAPPKEY',
          'secret': 'YOURAPPSECRET',
        }
      }
      'soundcloud': {
        'auth_params': {
          'display': 'window',
        },
        'creds': {
          'key': 'YOURAPPKEY',
          'secret': 'YOURAPPSECRET',
        }
      }
      'googleplus': {
        'app': 'myown.googleplus',
        'creds': {
          'key': 'YOURAPPKEY',
          'secret': 'YOURAPPSECRET',
        }
      }
    }

Views
-----
`social.auth.views` has three views which can be used both as views as well as
session manipulation functions.

`connect(request, callback_url=None, provider=settings.AUTH_DEFAULT_PROVIDER)`
Return a redirect url which will initialize an authentication request. The
given provider is stored in the session for calls to `callback`

`callback(request)`
Process authentication of credentials present on the request for the provider
stored in the session. On success, connects the user or the composite user to
the session. Returns a redirect to `request.GET['next']` if present, else `settings.LOGIN_REDIRECT_URL`.

`disconnect(request, provider=None)`
Calls the logout method of the given or active provider protocol.


Urls
----
Auths uses four urls which should be configured in the settings.

    # settings.py

    # These are the default values
    LOGIN_URL = '/connect/'
    LOGOUT_URL = '/disconnect/'
    LOGIN_CALLBACK_URL = '/callback/'
    LOGIN_REDIRECT_URL = '/'


Providers
---------
Auths comes with a slew of service providers, which provide models, parsers and tasks
to interact with their services.
