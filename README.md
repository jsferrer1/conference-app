# Conference Central App API Project

## Overview

Conference Central is an API server to support a provided conference organization that exists on the web as well as a native Android application. The API supports the following functionality: 

- User authentication (via Google accounts)
- User profiles
- Conference information
- Session information
- User wishlists
- Featured speaker

## Task 1: Design Choices

* Defined the following endpoint methods:
  - `getConferenceSessions(websafeConferenceKey)` -- Given a conference, return all sessions
  - `getConferenceSessionsByType(websafeConferenceKey, typeOfSession)` -- Given a conference, return all sessions of a specified type (eg lecture, keynote, workshop)
  - `getSessionsBySpeaker(speaker)` -- Given a speaker, return all sessions given by this particular speaker, across all conferences
  - `createSession(SessionForm, websafeConferenceKey)` -- open to the organizer of the conference

* Defined the Session and SessionForm class as:

  | Property        | Type             |
  |-----------------|------------------|
  | name            | string, required |
  | highlights      | string           |
  | speaker         | string, required |
  | duration        | integer          |
  | typeOfSession   | string           |
  | date            | date             |
  | startTime       | time             |
  | organizerUserId | string           |
  
* Defined the SpeakerForm class as:

   | Property        | Type                  |
   |-----------------|-----------------------|
   | speaker         | StringField           |
   | sessionNames    | StringField, repeated |

* Session and Speaker Design Choices
   
  Sessions were implemented using a parent-child design in order to cater to one-conference-to-many-session relationship. This allows for strong consistent querying, as sessions can be queried by their conference ancestor. The main reason for this design choice is speed and consistency between the conference and session entities. Many users would want to know about session information very often.
  
  On the other hand, speakers were implemented using a property of a particular session. Moreover, when sessions are added to the datastore, a memory cache (memcache) is created to contain the speaker and it's corresponding session names. This would enable the application to display featured speakers.

## Task 2: Add Sessions to User Wishlist

* Defined the following Endpoints methods:
  - `addSessionToWishlist(SessionKey)` -- adds the session to the user's list of sessions they are interested in attending
  - `getSessionsInWishlist()` -- query for all the sessions in a conference that the user is interested in

## Task 3: Work on indexes and queries

* Created the indexes on Session kind.

* Defined two additional query types:
  - `getSessionsToBeConfirmed()` -- Returns all sessions with missing duration/date/time information. This can be useful for administration and data integrity purposes.
  - `getUpcomingSessions()` -- Returns all upcoming sessions that is scheduled from today onwards.

* Query Problem
  - The problem related to the query is having one inequality filter on two different properties. With datastore, an inequality filter can be applied to at most one property only. If more than one property is filtered, it would result to `BadRequestError`.
  - To solve this problem, the following steps has been implemented:
    1. Using ndb, query for sessions that starts before 7pm (`startTime <= 19`)
    2. Using Python, manually filter those non-workshop sessions (`session.typeOfSession != 'workshop'`)

## Task 4: Add a Task

When a new session is added to a conference, the application will check the speaker. If there is more than one session by this speaker at this conference, added a new Memcache entry that features the speaker and session names. Memcache key is `FEATURED_SPEAKER`.

* Defined the following Endpoints methods:
  - `getFeaturedSpeaker()` -- Returns all the sessions of the featured speaker from the memcache. If no featured speaker exists in the memcache, it will show the next upcoming featured speaker from the datastore.
  
## Setup Instructions

To deploy this API server locally, ensure that you have downloaded and installed the [Google App Engine SDK for Python](https://cloud.google.com/appengine/downloads). Once installed, execute the following steps:

1. Download "conference.zip" and unzip the file into the folder `/vagrant/`.
2. (Optional) Update the value of `application` in `app.yaml` to the app ID you have registered in the App Engine admin console and would like to use to host your instance of this sample.
3. (Optional) Update the values at the top of `settings.py` to reflect the respective client IDs you have registered in the [Developer Console][4].
4. (Optional) Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
5. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080].)
6. (Optional) Generate your client library(ies) with [the endpoints tool][5].
7. (Optional) Deploy the application via `appcfg.py update`.  