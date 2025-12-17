// Enhanced Analytics Tracking System

// 1. EVENT TRACKING SERVICE (backend)
class AnalyticsService {
  // Track any user event with rich context
  async trackEvent(userId, eventName, properties = {}) {
    const enrichedData = {
      user_id: userId,
      event: eventName,
      properties: {
        ...properties,
        // Auto-enrich with context
        timestamp: new Date().toISOString(),
        day_of_week: new Date().getDay(),
        hour_of_day: new Date().getHours(),
        timezone: properties.timezone || 'UTC',
        session_id: properties.session_id,
        app_version: properties.app_version,
        platform: properties.platform
      }
    };

    // Send to multiple analytics providers
    await Promise.all([
      this.sendToRevenueCat(userId, eventName, properties),
      this.sendToMixpanel(enrichedData), // If using Mixpanel
      this.storeInDatabase(enrichedData), // Your own analytics DB
      this.sendToFirebase(enrichedData)  // Firebase Analytics
    ]);
  }

  // Store in your database for custom queries
  async storeInDatabase(data) {
    await models.UserEvent.create({
      userId: data.user_id,
      event: data.event,
      properties: JSON.stringify(data.properties),
      createdAt: new Date()
    });
  }
}

// 2. KEY EVENTS TO TRACK

// Onboarding Flow
trackEvent(userId, 'onboarding_started');
trackEvent(userId, 'phone_verification_attempted', { method: 'SMS' });
trackEvent(userId, 'phone_verification_completed', { attempts: 2, duration_seconds: 45 });
trackEvent(userId, 'self_recipient_created');
trackEvent(userId, 'onboarding_completed', { total_duration_seconds: 180 });

// Message Creation Flow
trackEvent(userId, 'message_draft_started', { 
  recipient_type: 'self',
  entry_point: 'home_screen' 
});
trackEvent(userId, 'audio_recording_started');
trackEvent(userId, 'audio_recording_completed', { 
  duration_seconds: 45,
  file_size_kb: 234 
});
trackEvent(userId, 'message_saved', { 
  has_audio: true,
  has_photo: false,
  word_count: 125 
});

// User Engagement
trackEvent(userId, 'app_opened', { 
  time_since_last_open_hours: 18,
  notification_triggered: false 
});
trackEvent(userId, 'daily_active', { 
  consecutive_days: 7,
  total_days_active: 45 
});
trackEvent(userId, 'message_listened', { 
  message_age_days: 30,
  completion_percentage: 100 
});

// Feature Usage
trackEvent(userId, 'premium_feature_attempted', { 
  feature: 'unlimited_messages',
  user_plan: 'free' 
});
trackEvent(userId, 'share_initiated', { 
  method: 'invite_link',
  share_target: 'family' 
});

// 3. SESSION TRACKING
class SessionTracker {
  async startSession(userId, deviceId) {
    const session = {
      id: generateUUID(),
      userId,
      deviceId,
      startTime: new Date(),
      events: []
    };
    
    // Store in Redis for real-time access
    await redis.setex(`session:${session.id}`, 3600, JSON.stringify(session));
    
    return session.id;
  }

  async endSession(sessionId) {
    const session = await redis.get(`session:${sessionId}`);
    if (session) {
      const data = JSON.parse(session);
      const duration = new Date() - new Date(data.startTime);
      
      // Store completed session
      await models.UserSession.create({
        ...data,
        endTime: new Date(),
        durationSeconds: Math.floor(duration / 1000),
        eventCount: data.events.length
      });
    }
  }
}

// 4. FUNNEL ANALYSIS QUERIES
const funnelQueries = {
  // Onboarding funnel
  async getOnboardingFunnel(startDate, endDate) {
    return await sequelize.query(`
      WITH funnel AS (
        SELECT 
          user_id,
          MAX(CASE WHEN event = 'signup_completed' THEN 1 ELSE 0 END) as signed_up,
          MAX(CASE WHEN event = 'phone_verification_completed' THEN 1 ELSE 0 END) as verified_phone,
          MAX(CASE WHEN event = 'self_recipient_created' THEN 1 ELSE 0 END) as created_recipient,
          MAX(CASE WHEN event = 'first_message_created' THEN 1 ELSE 0 END) as created_message
        FROM user_events
        WHERE created_at BETWEEN :startDate AND :endDate
        GROUP BY user_id
      )
      SELECT 
        COUNT(*) as total_users,
        SUM(signed_up) as signed_up,
        SUM(verified_phone) as verified_phone,
        SUM(created_recipient) as created_recipient,
        SUM(created_message) as created_message,
        ROUND(100.0 * SUM(verified_phone) / SUM(signed_up), 2) as verification_rate,
        ROUND(100.0 * SUM(created_message) / SUM(signed_up), 2) as activation_rate
      FROM funnel
    `, { 
      replacements: { startDate, endDate },
      type: QueryTypes.SELECT 
    });
  }
};

// 5. COHORT ANALYSIS
class CohortAnalytics {
  async getRetentionCohorts() {
    return await sequelize.query(`
      WITH cohorts AS (
        SELECT 
          DATE_TRUNC('week', u.created_at) as cohort_week,
          u.id as user_id,
          DATE_TRUNC('week', e.created_at) as event_week,
          COUNT(DISTINCT DATE(e.created_at)) as active_days
        FROM users u
        LEFT JOIN user_events e ON u.id = e.user_id
        WHERE e.event = 'daily_active'
        GROUP BY 1, 2, 3
      )
      SELECT 
        cohort_week,
        COUNT(DISTINCT user_id) as cohort_size,
        COUNT(DISTINCT CASE WHEN event_week = cohort_week THEN user_id END) as week_0,
        COUNT(DISTINCT CASE WHEN event_week = cohort_week + INTERVAL '1 week' THEN user_id END) as week_1,
        COUNT(DISTINCT CASE WHEN event_week = cohort_week + INTERVAL '2 weeks' THEN user_id END) as week_2,
        COUNT(DISTINCT CASE WHEN event_week = cohort_week + INTERVAL '3 weeks' THEN user_id END) as week_3,
        COUNT(DISTINCT CASE WHEN event_week = cohort_week + INTERVAL '4 weeks' THEN user_id END) as week_4
      FROM cohorts
      GROUP BY cohort_week
      ORDER BY cohort_week DESC
    `);
  }
}