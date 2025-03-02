## Once on a gloomy winter morning

I woke up, brewed myself a pot of coffee, opened EKNM website and got `ERR_SSL_PROTOCOL_ERROR`. That was rather unexpected, as I recently refreshed the certificates for all subdomains and didn't change anything in last few weeks. After some time and contless experiments I concluded following symptoms:
- There are continuous periods of server returning the error
- Periods mostly last for 10 - 30 minutes
- There's no patterns of period start/end times (at least I could not notice any)
- Periods don't depend on what subdomain and certificates are used
- Nginx container logs don't show any errors

My wild guess was some kind of resource leak, where periods appear when some connection pool gets exhausted. Then some kind of garbage collector runs and fixes the error.
While thinking about this theory I understood that I know nothing about how SSL certificates work, both from network and server perspective. So let's dive into it.