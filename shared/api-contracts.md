# LittleTalks API Endpoints & Entitlement Flow

## API Endpoints

### 1. **Authentication Endpoints**
- `POST /api/auth/signUp` - Create new account
- `POST /api/auth/signIn` - Login with email/password  
- `POST /api/auth/google` - Google OAuth login
- `POST /api/auth/apple` - Apple OAuth login
- `POST /api/auth/signOut` - Logout
- `POST /api/auth/forgotPassword` - Password reset
- `POST /api/auth/passcodeVerify` - Verify phone with passcode
- `POST /api/auth/verifyOTP` - Verify OTP for password reset

### 2. **User Endpoints**
- `GET /api/user/` - Get current user info
- `PUT /api/user/` - Update user profile
- `GET /api/user/limits?revenueCatEntitlement=2|8|2|m` - Get user limits based on subscription
- `POST /api/user/device` - Register device for push notifications

### 3. **Character (Persona) Endpoints**
- `GET /api/persona/` - Get all characters for user
- `POST /api/persona/` - Create new character (checks entitlements)
- `GET /api/persona/:id` - Get specific character
- `PUT /api/persona/:id` - Update character
- `DELETE /api/persona/:id` - Delete character

### 4. **Recipient Endpoints (Unified System)**
- `GET /api/receiver/` - Get all recipients (version aware - filters isMe for older apps)
- `POST /api/receiver/` - Create new recipient (no phone verification required)
- `GET /api/receiver/:id` - Get specific recipient
- `PUT /api/receiver/:id` - Update recipient
- `DELETE /api/receiver/:id` - Delete recipient (prevents deleting self-recipients)
- `GET /api/sender/` - Get senders (auto-creates self-recipient if none exists)
- `POST /api/sender/` - Create verified sender (requires SMS verification)
- `POST /api/sender/request` - Request SMS verification code

### 5. **Message Endpoints**
- `GET /api/message/` - Get all messages
- `POST /api/message/` - Send new message (checks entitlements)
- `GET /api/message/:id` - Get specific message
- `DELETE /api/message/:id` - Delete message

### 6. **Conversation Endpoints**
- `GET /api/conversation/` - Get all conversations
- `POST /api/conversation/` - Create new conversation
- `GET /api/conversation/:id` - Get specific conversation
- `PUT /api/conversation/:id` - Update conversation
- `DELETE /api/conversation/:id` - Delete conversation

### 7. **Configuration Endpoints**
- `GET /api/freeTierConfiguration/` - Get current free tier limits (public endpoint)

### 8. **Other Endpoints**
- `GET /api/` - Health check
- `GET /api/version` - Get API version info
- `POST /api/feedback` - Submit user feedback
- `GET /api/notification/` - Get notifications

---

## Entitlement Flow Visualization - ACTUAL IMPLEMENTATION

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CHARACTER CREATION FLOW                       │
└─────────────────────────────────────────────────────────────────────┘

    USER TAPS "+" BUTTON
           │
           ▼
    ┌──────────────┐
    │ Characters.js │
    └──────┬───────┘
           │ navigation.navigate("CharacterName")
           ▼
    ┌──────────────────┐
    │ CharacterName.js │ ─── User enters character details
    └──────┬───────────┘
           │ 
           ▼
    ┌──────────────────┐
    │ CharacterDob.js  │ ─── User enters DOB (optional)
    └──────┬───────────┘
           │ onNext()
           ▼
    ┌──────────────────┐
    │  createPersona   │ ─── API call to POST /api/persona/
    │   (persona.js)   │     {name, revenueCatEntitlement}
    └──────┬───────────┘
           │
           ▼
    ┌─────────────────────────────────────────────────────┐
    │        BACKEND ENTITLEMENT CHECK (persona.js)        │
    │                                                       │
    │  1. Extract userId from JWT token (req.auth.userId)  │
    │  2. Get revenueCatEntitlement from request body      │
    │  3. Determine if user has subscription               │
    └───────────────────┬─────────────────────────────────┘
                        │
        ┌───────────────┴───────────────┐
        │                               │
        ▼                               ▼
   HAS SUBSCRIPTION?              FREE TIER USER?
        │                               │
        ▼                               ▼
┌──────────────────────┐      ┌──────────────────────────┐
│ Get subscription     │      │ Get current free tier    │
│ details from         │      │ configuration from DB    │
│ RevenueCat           │      │ (getCurrentFreeTierDetails) │
└──────────┬───────────┘      └───────────┬──────────────┘
           │                               │
           ▼                               ▼
┌──────────────────────┐      ┌──────────────────────────┐
│ Parse entitlement    │      │ Calculate period based   │
│ identifier to get    │      │ on free tier start date  │
│ character limit      │      │ and renewal period       │
│ (configs.getLimit)   │      │                          │
└──────────┬───────────┘      └───────────┬──────────────┘
           │                               │
           ▼                               ▼
┌──────────────────────┐      ┌──────────────────────────┐
│ Count characters     │      │ Count characters created │
│ created after last   │      │ in current free tier     │
│ subscription renewal │      │ period with              │
│ with billingSource:  │      │ billingSource:           │
│ 'subscription'       │      │ 'free_tier'              │
└──────────┬───────────┘      └───────────┬──────────────┘
           │                               │
           └──────────────┬────────────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │ Check if count   │
                 │ >= limit         │
                 └──────┬──────────┘
                        │
         ┌──────────────┴──────────────┐
         │                             │
         ▼                             ▼
    COUNT < LIMIT                 COUNT >= LIMIT
         │                             │
         ▼                             ▼
┌──────────────────────┐      ┌──────────────────────────┐
│ Assign random phone  │      │ throw new Error          │
│ number from pool     │      │ ("MAX_CHARACTERS_REACHED")│
│ & create character   │      │                          │
└──────────────────────┘      └──────────────────────────┘
         │                             │
         ▼                             ▼
    SUCCESS RESPONSE              ERROR RESPONSE
```

## Complete Entitlement Flow - Frontend to Backend

### CHARACTER CREATION FLOW (CharacterName.js)
```javascript
// 1. User enters character name and taps Next
// CharacterName.js - No entitlement check here
navigation.navigate("Character Icon", { name });

// 2. Character Icon screen (not shown) - User selects avatar
// 3. CharacterDob.js - User enters DOB (optional)
// 4. Final submission calls createPersona API

// From mobile app (persona.js API):
const response = await API.postWithAuth("persona/", {
  name: characterName,
  revenueCatEntitlement: revenueCatEntitlement // From user context
});
```

### RECIPIENT CREATION FLOW (RecipientName.js)
```javascript
// Frontend checks count from context before API call
const currentRecipients = conversation.child || [];
const nonSelfRecipients = currentRecipients.filter(r => !r.isMe);

if (nonSelfRecipients.length >= senderLimit) {
  BasicAlert(t("Limit Reached"), t("You've reached your recipient limit"));
  return;
}

// Create via receiver endpoint (no SMS verification)
const createResp = await API.postWithAuth("receiver/", {
  firstName,
  lastName,
  phoneNumber: phonePlain,
  countryCode: country.dial_code,
  revenueCatEntitlement: revenueCatEntitlement,
  billingSource: revenueCatEntitlement ? 'subscription' : 'free_tier'
});
```

### MESSAGE CREATION FLOW
```javascript
// From mobile app message API
const response = await API.postWithAuth("message/", {
  ConversationId,
  message,
  type,
  attachment,
  revenueCatEntitlement: revenueCatEntitlement
});
```

---

## Key Implementation Details

### Frontend (Mobile App)
1. **No blocking validation** on navigation - better UX
2. **userContext** provides limits and current usage
3. **RevenueCat SDK** manages subscriptions
4. **Error handling** navigates to subscription screen on limit reached

### Backend (API)
1. **Secure validation** - all entitlement checks server-side
2. **Billing period aware** - counts only current period for subscriptions
3. **Free tier support** - tracks remaining benefits via `getRemainingFreeTierBenefits()`
4. **Graceful degradation** - returns appropriate error messages

### Key Functions in configs.js:
- `getLimit()` - Retrieves configuration limits based on type and entitlement
- `getAllMessages()` - Gets all messages for a user
- `getAllCharacters()` - Gets all characters (personas) for a user  
- `getCurrentFreeTierDetails()` - Gets current free tier configuration
- `getRemainingFreeTierBenefits()` - Calculates remaining free tier allowances

## How Entitlement Checking Actually Works

### 1. **Characters/Personas** (from persona.js controller)
```javascript
// When creating a character:
if (userHasSubscription) {
  // Get limit from config based on RevenueCat identifier
  const limit = await configs.getLimit("characterLimit", subscriptionId);
  charactersLimit = Number.parseInt(limit[0].number);
  
  // Count characters created AFTER last subscription renewal
  charactersQuery = {
    userId,
    status: "active",
    createdAt: { [Op.gte]: lastRenewalDate },
    billingSource: "subscription"
  };
} else {
  // Free tier: get current configuration
  const freeTierDetails = await configs.getCurrentFreeTierDetails();
  charactersLimit = Number.parseInt(freeTierDetails.characters);
  
  // Calculate current free tier period
  // Count characters in current period only
  charactersQuery = {
    userId,
    createdAt: { [Op.gte]: freeTierLastRenewalDate },
    status: "active",
    billingSource: "free_tier"
  };
}

const personas = await models.Persona.findAll({where: charactersQuery});
if (personas.length >= charactersLimit) {
  throw new Error("MAX_CHARACTERS_REACHED");
}
```

### 2. **Messages** (from message.js controller)
```javascript
// Similar pattern but messages reset each period:
if (userHasSubscription) {
  messagesLimit = Number.parseInt(limit[0].number);
  messagesQuery = {
    UserId: userId,
    createdAt: { [Op.gte]: lastRenewalDate },
    billingSource: 'subscription'
  };
} else {
  // Free tier calculation similar to characters
  messagesQuery = {
    UserId: userId,
    createdAt: { [Op.gte]: freeTierLastRenewalDate },
    billingSource: 'free_tier'
  };
}

const messages = await models.Message.findAll({where: messagesQuery});
if (messages.length >= messagesLimit) {
  throw new Error("MAX_MESSAGES_REACHED");
}
```

### 3. **Recipients/Senders** (from sender.js controller)
```javascript
// Recipients use the same pattern:
if (userHasSubscription) {
  sendersLimit = limit[0].number;
  sendersQuery = {
    UserId,
    billingSource: 'subscription',
    createdAt: { [Op.gte]: lastRenewalDate }
  };
} else {
  sendersLimit = Number.parseInt(freeTierDetails.senders);
  sendersQuery = {
    UserId,
    createdAt: { [Op.gte]: freeTierLastRenewalDate },
    billingSource: 'free_tier'
  };
}

const senders = await models.Sender.findAll({ where: sendersQuery });
if (senders.length >= sendersLimit) {
  throw new Error("MAX_SENDERS_REACHED");
}
```

---

## RevenueCat Entitlement Format

### Subscription Entitlement String: `2|8|2|m`
- **Position 1**: Recipients limit (2)
- **Position 2**: Messages per MONTH (8) - Always monthly regardless of billing interval
- **Position 3**: Characters/Personas limit (2)
- **Position 4**: Billing interval (m=monthly, y=yearly) - Only affects payment frequency

### Example Entitlements:
- `2|8|2|m` - 2 recipients, 8 messages/month, 2 characters, billed monthly
- `4|20|5|m` - 4 recipients, 20 messages/month, 5 characters, billed monthly
- `10|100|10|y` - 10 recipients, 100 messages/month, 10 characters, billed yearly (saves money)

### Important Notes:
- **Messages are ALWAYS per month** for subscriptions, regardless of billing cycle
- Yearly billing is a discount option but limits remain monthly
- Characters and recipients are cumulative (don't reset)

## Free Tier System

### How Free Tier Works
Free tier users get limited access without payment to encourage app exploration:

1. **Configuration**: Stored in FreeTierConfiguration table
2. **Period**: Can be different from monthly (e.g., "P1W" for weekly, "P2W" for bi-weekly)
3. **Limits Reset**: Based on the period specified in configuration
4. **Tracking**: Uses `billingSource: 'free_tier'` to distinguish from paid items

### Free Tier Period Calculation (from code):
```javascript
// From persona.js controller
const freeTierPeriod = freeTierDetails.period; // e.g., "P1M", "P1W", "P2W"
const freeTierBeginDate = freeTierDetails.startDate;
const now = currentDate();
const timeSinceFreeTierBeginning = now.getTime() - freeTierBeginDate.getTime();
const freeTierPeriodMilliseconds = dayjs.duration(freeTierPeriod).asMilliseconds();
const freeTierLastRenewalTime = freeTierBeginDate.getTime() + 
  Math.floor(timeSinceFreeTierBeginning / freeTierPeriodMilliseconds) * 
  freeTierPeriodMilliseconds;
```

### Free Tier vs Subscription
| Feature | Free Tier | Subscription |
|---------|-----------|--------------|
| Message Reset | Per configured period (weekly/bi-weekly/monthly) | Always monthly |
| Character Limit | Cumulative from period start | Cumulative from subscription start |
| Recipient Limit | Cumulative from period start | Cumulative from subscription start |
| Billing Source | 'free_tier' | 'subscription' |
| Configuration | FreeTierConfiguration table | Config table with RevenueCat ID |

### Free Tier Strategy
- Gives users a taste of the app's capabilities
- More restrictive periods (weekly) create urgency
- Lower limits encourage upgrading
- No payment info required to start

## Database Schema Details

### Config Table
```sql
Config {
  type: STRING,              // "messageLimit", "characterLimit", "senderLimit"
  number: INTEGER,           // The limit value
  revenueCatEntitlement: STRING  // e.g., "2|8|2|m"
}
```

### FreeTierConfiguration Table
```sql
FreeTierConfiguration {
  id: INTEGER PRIMARY KEY,
  enabled: BOOLEAN,
  startDate: DATE,           // When this config became active
  endDate: DATE,             // NULL for current active config
  period: STRING,            // ISO 8601 duration (e.g., "P1M" for 1 month)
  characters: INTEGER,       // Character limit for free tier
  messages: INTEGER,         // Message limit for free tier
  senders: INTEGER          // Recipient limit for free tier
}
```

### Entity Tables (Persona, Message, Sender/Receiver)
All include:
- `billingSource`: ENUM('subscription', 'free_tier')
- `createdAt`: TIMESTAMP (for period calculations)
- `UserId`: Foreign key to User table

### Recipients Table (unified for Sender & Receiver models)
- Both models point to same `recipients` table
- `isMe`: BOOLEAN flag to identify self-recipients
- `alias`: STRING for display name (e.g., "Me")
- Self-recipients auto-created on first sender fetch