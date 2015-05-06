# ud858-final
final project for udacity ud858 course

## Products
- [App Engine][1]

## Language
- [Python][2]

## APIs
- [Google Cloud Endpoints][3]

## Setup Instructions
1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host
   your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][4].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][5].)
1. (Optional) Generate your client library(ies) with [the endpoints tool][6].
1. Deploy your application.

## Task 1: Add Sessions to a Conference
####Explain your design choices
For session implementation I largely follow the instruction by defining a Session ndb.Model and a SessionForm message object. I also wrote several helper functions similar to help retrieving and creating session and speaker records

For speaker implementation I decided to use speaker name as a string to represent a speaker, so no speaker model is created. This approach has several severe flaws such as name collision and you can't store additional info about the speaker. The biggest advantage is that it is dead simple to implement, and this is why i am taking this approach. Should this approach really become a problem, we can always change it in V2.

##Task 3: Work on indexes and queries
####Come up with 2 additional queries
```
# conference that took place in SF or Tokyo
conferences = Conference.query(ndb.OR(Conference.city == 'Tokyo', 
                                      Conference.city == 'Paris'))
```

```
# top 5 most anticipated conferences (by # of wishlisted sessions)
@ndb.tasklet
def callback(conference):
    count = yield WishlistSession.query(WishlistSession.sessionConferenceKey == conference.key.urlsafe()).count_async()
    raise ndb.Return((count, conference))

for count, conf in sorted(Conference.query().map(callback), key=lambda tup: tup[0], reverse=True)[:5]:
    print '{} - ({})'.format(conf.name, count)
```

####Solve the following query related problem
```
seven_pm = datetime.datetime.strptime("19:00", "%H:%M").time()
anything_but_workshop = ['NOT_SPECIFIED', 'LECTURE', 'KEYNOTE']
for session in Session.query(ndb.AND(Session.typeOfSession.IN(anything_but_workshop), 
                                     Session.startTime < seven_pm)):
    print '{} - {} - {}'.format(session.name, session.typeOfSession, 
                                session.startTime.strftime("%H:%M"))
```
This is one way I can think of to accomplish said requirement. The main obstacle is that this query requires two inequalities comparsion that is not allowed by ndb api.

I would also try just filter out sessions after 7pm, and then iterate the results to remove any WORKSHOP sessions. I would then use benchmark tools like AppStats to compare the performance.

This kind of complexity queries with multiple inequality operators and/or groups, is probably best to use MapReduce to solve.

[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://console.developers.google.com/
[5]: https://localhost:8080/
[6]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool
