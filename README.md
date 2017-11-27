# sequelize-associations
Friend requests, group subscriptions, invitations using Sequelize associations

## Docs
Sequelize documentation on [associations](http://docs.sequelizejs.com/manual/tutorial/associations.html)

Getters and Setters found [here](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#getters-setters)

BelongsTo, BelongsToMany found [here](http://docs.sequelizejs.com/class/lib/associations/belongs-to.js~BelongsTo.html)

## [Demo](https://gangpay.herokuapp.com/)
Demo site to view friend requests, group subscriptions, team members join, etc.

Also check out the [production site](https://ganpay.herokuapp.com/).

## Sequelize Associations

### Basic Business Goal
- Users can send friend requests to other users. 
- Users can create teams to which they can invite friends as members.
- A user can view a list of: friends, teams(subscriptions), and each team's members.
- Only friends can access to friends' profiles, and only members can access to their teams.

### Design
- User-to-Team relationship:  A user can create many teams(subscriptions), each containing many users(members).
- User-to-User relationship:  A user(requester) can send a friend request to another user(requestee). If the requestee confirms, they will be friends.


### Models
- User
- Team
- (Will skip other models for now)

```js
//USER
classMethods: {
  associate(models) {
    User.belongsToMany(models.Team, { as: 'Subscriptions', through: 'user_teams', foreignKey: User.id });
    User.belongsToMany(User, { as: 'Friends', through: 'friends' });
    User.belongsToMany(User, { as: 'Requestees', through: 'friendRequests', foreignKey: 'requesterId', onDelete: 'CASCADE'});
    User.belongsToMany(User, { as: 'Requesters', through: 'friendRequests', foreignKey: 'requesteeId', onDelete: 'CASCADE'});
  }
}
```

```js
//TEAM
Team.belongsToMany(models.User, { as: 'Members', through: 'user_teams', foreignKey: Team.id });
```

### Routes
```js
//USER
export default function(app, express, router) {
  router.param('id', usersController.setUserById);
  router.get('/user/:id', usersController.getUserById);
  ...
  router.put('/user/:id/subscription', usersController.putSubscription);
  router.get('/user/:id/subscription', usersController.getSubscription);
  ...
  router.put('/user/:id/friendRequest', usersController.putFriendRequest);
  router.get('/user/:id/friendRequestReceived', usersController.getFriendRequestReceived);
  router.get('/user/:id/friendRequestSent', usersController.getFriendRequestSent);
  ...
  router.put('/user/:id/acceptFriendRequest', usersController.acceptFriendRequest);
  router.get('/user/:id/friends', usersController.getFriends);
  router.put('/user/:id/cancelFriendRequest', usersController.cancelFriendRequest);
  router.put('/user/:id/unfriend', usersController.unfriend);
}
```

```js
//TEAM
export default function(app, express, router) {
  router.param('teamId', teamsController.setTeamById);
  router.get('/team/:teamId', teamsController.getTeamById);
  router.put('/team/:teamId/members', teamsController.putMembers);
  router.get('/team/:teamId/members', teamsController.getMembers);
  router.get('/team/:teamId/member', teamsController.getMemberById);
  ...
}
```

### Controllers
```js
//USER
setUserById(req, res, next, id) {
  console.log('Set router.param for :userId');
  db.User.findById(id)
    .then(user => {
      if (!user) res.sendStatus(404);
      else {
        req.user = user;
        next();
      }
    })
    .catch(next);
},

getUserById(req, res) {
  console.log("Get user by id");
  res.send(req.user);
},

putSubscription(req, res, next) {
  console.log('Subscribe user to the team');
  req.user.addSubscriptions(req.body.teamId)
    .then(result => res.status(201).send(result))
    .catch(next);
},

getSubscription(req, res, next) {
  console.log('Get all teams the user is subscribed to');
  req.user.getSubscriptions()
    .then(result => res.status(201).send(result))
    .catch(next);
},

putFriendRequest(req, res, next) {
  if (req.body.requesteeId != req.user.id) {
    console.log('Send friend request');
    req.user.addRequestees(req.body.requesteeId)
      .then(result => res.status(201).send(result))
      .catch(next);
  } else {
    res.status(400).send('Cannot friend yourself');
  }
},
...
```

```js
//TEAM
setTeamById(req, res, next, id) {
  console.log('Set router.param for :teamId');
  db.Team.findById(id)
    .then(team => {
      if (!team) res.sendStatus(404);
      else {
        req.team = team;
        next();
      }
    })
    .catch(next);
},

getTeamById(req, res) {
  console.log("Get team by id");
  res.send(req.team);
},

putMembers(req, res, next) {
  console.log('Add members to team');
  req.team.addMembers(req.body.friendIds)
    .then(result => res.status(201).send(result))
    .catch(next);
},

getMembers(req, res, next) {
  console.log('Get team members');
  req.team.getMembers()
    .then(users => {
      var updated = users.map((user) => {
        return {
          id: user.id,
          username: user.username,
          name: `${user.first_name} ${user.last_name}`
        }
      });
      res.status(201).send(updated);
    })
    .catch(next);
},
...
 ```
