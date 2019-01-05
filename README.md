# JobsFinder
A script which parses Upwork RSS feed and sends notifications to Telegram.

```js
const credentials = require('./credentials.json')

const FeedSub = require('feedsub')
const rssFeed = credentials.rss_feed
let reader = new FeedSub(rssFeed, {
  interval: 1 // Check feed every 1 minute.
})

const low = require('lowdb')
const FileSync = require('lowdb/adapters/FileSync')
const adapter = new FileSync('db.json')
const db = low(adapter)

db.defaults({ feed: [] }).write()

const Telegraf = require('telegraf')
const Extra = require('telegraf/extra')
const session = require('telegraf/session')
const token = credentials.telegram_bot_token
const bot = new Telegraf(token)

// Register session middleware.
bot.use(session())

// Register logger middleware.
bot.use((ctx, next) => {
  const start = new Date()
  return next().then(() => {
    const ms = new Date() - start
    console.log('response time %sms', ms)
  })
})

reader.on('item', (item) => {
  console.log(item.title)

  const itemInDb = db.get('feed').find({ link: item.link }).value()
  if (itemInDb) {
    console.log("This item is already exists:")
    console.log(itemInDb.link)
  } else {
    db.get('feed').push(item).write()
    var message = item.description
    bot.telegram.sendMessage(credentials.telegram_channel, message, Extra.HTML().markup())
  }
})

reader.start()

```
