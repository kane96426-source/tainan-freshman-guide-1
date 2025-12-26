function doGet(e) {
  var page = e.parameter.page || 'index';
  return HtmlService.createTemplateFromFile(page).evaluate()
    .addMetaTag('viewport', 'width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL)
    .setTitle("南大魔法學院 - 史詩傳奇指南");
}

function getScriptUrl() { return ScriptApp.getService().getUrl(); }

// --- 1. 核心驗證與註冊 ---
function login(u, p) {
  var data = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Users").getDataRange().getValues();
  for (var i = 1; i < data.length; i++) {
    if (data[i][0].toString().trim() === u.toString().trim() && data[i][1].toString().trim() === p.toString().trim()) {
      return { status: "success", nickname: data[i][2], uid: data[i][4] };
    }
  }
  return { status: "fail" };
}

function registerUser(u, p, n) {
  var ss = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Users");
  if (ss.getDataRange().getValues().some(r => r[0].toString() === u)) return "此編號已被契約佔用";
  var uid = Math.random().toString(36).substring(2, 7).toUpperCase();
  var avatar = "https://api.dicebear.com/7.x/adventurer/svg?seed=" + encodeURIComponent(n);
  ss.appendRow([u, p, n, avatar, uid]);
  return "契約成立！UID: " + uid;
}

// --- 2. 匿名大釜系統 (CRUD: Create/Read/Update/Delete) ---
function getPostsAndReplies() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var posts = ss.getSheetByName("Posts").getDataRange().getValues();
  var reps = ss.getSheetByName("Replies").getDataRange().getValues();
  posts.shift(); reps.shift();
  var replies = reps.map(r => ({ post_id: r[0], nickname: r[1], content: r[2], time: r[3].toString() }));
  return posts.map(r => ({
    id: r[0], nickname: r[1], content: r[2], image: r[3], category: r[4], likes: parseInt(r[5] || 0), time: r[6].toString(),
    replies: replies.filter(rep => rep.post_id == r[0])
  }));
}

function addPost(nick, content, imgData, cat) {
  SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Posts").appendRow(["P"+Date.now(), nick, content, imgData, cat, 0, new Date()]);
}

function deletePost(id, nick) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Posts");
  var data = sheet.getDataRange().getValues();
  for(var i=1; i<data.length; i++) {
    if(data[i][0] == id && data[i][1] == nick) { sheet.deleteRow(i+1); return "OK"; }
  }
}

function addReply(postId, nick, content) {
  SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Replies").appendRow([postId, nick, content, new Date()]);
}

function likePost(postId, userUid) {
  var lock = LockService.getScriptLock();
  try {
    lock.waitLock(10000);
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    if (ss.getSheetByName("Likes").getDataRange().getValues().some(r => r[0] == postId && r[1] == userUid)) return { status: "error" };
    ss.getSheetByName("Likes").appendRow([postId, userUid]);
    var postSheet = ss.getSheetByName("Posts");
    var posts = postSheet.getDataRange().getValues();
    for (var j = 1; j < posts.length; j++) if (posts[j][0] == postId) { postSheet.getRange(j+1, 6).setValue(parseInt(posts[j][5]||0)+1); return {status:"success"}; }
  } finally { lock.releaseLock(); }
}

// --- 3. 美食羅盤系統 (CRUD: Read/Update/Delete) ---
function getRestaurants() {
  var data = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Restaurants").getDataRange().getValues();
  data.shift();
  return data.map(r => {
    var cms = []; try { cms = r[7] ? JSON.parse(r[7]) : []; } catch(e) { cms = []; }
    return { id: r[0], name: r[1], lat: r[2], lng: r[3], desc: r[4], avgStars: r[5] || 0, address: r[6], comments: cms };
  });
}

function addRestaurantReview(shopId, userNick, star, text) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Restaurants");
  var data = sheet.getDataRange().getValues();
  for (var i = 1; i < data.length; i++) {
    if (data[i][0] == shopId) {
      var reviews = []; try { reviews = data[i][7] ? JSON.parse(data[i][7]) : []; } catch(e) { reviews = []; }
      reviews.unshift({ user: userNick, star: parseInt(star), text: text, time: Date.now() });
      var avg = (reviews.reduce((sum, r) => sum + r.star, 0) / reviews.length).toFixed(1);
      sheet.getRange(i+1, 6).setValue(avg);
      sheet.getRange(i+1, 8).setValue(JSON.stringify(reviews));
      return { status: "success", newAvg: avg };
    }
  }
}

function deleteRestaurantReview(sid, userNick, ts) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Restaurants");
  var data = sheet.getDataRange().getValues();
  for (var i = 1; i < data.length; i++) {
    if (data[i][0] == sid) {
      var revs = JSON.parse(data[i][7] || "[]");
      var newRevs = revs.filter(r => !(r.user === userNick && Number(r.time) === Number(ts)));
      var avg = newRevs.length > 0 ? (newRevs.reduce((sum, r) => sum + r.star, 0) / newRevs.length).toFixed(1) : 0;
      sheet.getRange(i+1, 6).setValue(avg);
      sheet.getRange(i+1, 8).setValue(JSON.stringify(newRevs));
      return "OK";
    }
  }
}

// --- 4. 課表與羈絆系統 ---
function saveSchedule(uid, data) {
  var ss = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Schedules");
  var rows = ss.getDataRange().getValues();
  for(var i=1; i<rows.length; i++) if(rows[i][1].toString() === uid) { ss.getRange(i+1, 3).setValue(JSON.stringify(data)); return; }
  ss.appendRow(["", uid, JSON.stringify(data)]);
}

function getFriendSchedule(fuid) {
  var data = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Schedules").getDataRange().getValues();
  for(var i=1; i<data.length; i++) if(data[i][1].toString() === fuid) return JSON.parse(data[i][2]);
  return null;
}

function addMagicFriend(myUid, fuid) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var userData = ss.getSheetByName("Users").getDataRange().getValues();
  if(!userData.some(u => u[4].toString() === fuid)) return {status:"error", msg:"查無巫師"};
  ss.getSheetByName("Friends").appendRow([myUid, fuid]);
  return {status:"success"};
}

function getFriendsListInfo(myUid) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var rels = ss.getSheetByName("Friends").getDataRange().getValues();
  var users = ss.getSheetByName("Users").getDataRange().getValues();
  var schedules = ss.getSheetByName("Schedules").getDataRange().getValues();
  var fids = [];
  for(var i=1; i<rels.length; i++) if(rels[i][0] == myUid) fids.push(rels[i][1]);
  return fids.map(fuid => {
    var uRow = users.find(u => u[4] == fuid);
    var sRow = schedules.find(s => s[1] == fuid);
    return { uid: fuid, nickname: uRow?uRow[2]:"未知", schedule: sRow?JSON.parse(sRow[2]):{} };
  });
}

function removeFriend(myUid, fuid) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Friends");
  var data = sheet.getDataRange().getValues();
  for (var i = 1; i < data.length; i++) if (data[i][0] == myUid && data[i][1] == fuid) { sheet.deleteRow(i + 1); return "OK"; }
}
