# LittleTalks Intelligent Journey System
## Technical Implementation Guide

---

## 🏗️ **System Architecture Overview**

### **High-Level Architecture**
```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Mobile App    │    │  Backend API     │    │   Analytics     │
│                 │    │                  │    │                 │
│ ┌─────────────┐ │    │ ┌──────────────┐ │    │ ┌─────────────┐ │
│ │Journey Hook │ │◄──►│ │Journey Engine│ │◄──►│ │Event Store  │ │
│ └─────────────┘ │    │ └──────────────┘ │    │ └─────────────┘ │
│ ┌─────────────┐ │    │ ┌──────────────┐ │    │ ┌─────────────┐ │
│ │ChoiceModal  │ │    │ │Sentiment API │ │    │ │ ML Pipeline │ │
│ └─────────────┘ │    │ └──────────────┘ │    │ └─────────────┘ │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

### **Core Components**

1. **Journey Orchestrator** - Central intelligence system
2. **Sentiment Analysis Engine** - Message emotion detection  
3. **Cross-Feature Intelligence** - Pattern recognition across features
4. **Contextual Intelligence** - Environmental awareness
5. **Churn Prediction Engine** - Risk assessment and prevention
6. **Real-Time Learning** - Continuous system improvement

---

## 📊 **Database Schema Design**

### **New Tables Required**

```sql
-- ===================================================================
-- JOURNEY SYSTEM CORE TABLES
-- ===================================================================

-- Journey step definitions (configurable without code deploys)
CREATE TABLE journey_steps (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  step_key VARCHAR(64) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  title VARCHAR(255) NOT NULL,
  message TEXT NOT NULL,
  primary_action VARCHAR(100) NOT NULL,
  secondary_action VARCHAR(100) NOT NULL,
  priority INTEGER NOT NULL DEFAULT 100,
  category VARCHAR(50) NOT NULL,
  destination_screen VARCHAR(100),
  destination_params JSONB DEFAULT '{}',
  trigger_config JSONB NOT NULL,
  cooldown_hours INTEGER DEFAULT 24,
  max_dismissals INTEGER DEFAULT 3,
  expires_after_days INTEGER,
  is_active BOOLEAN DEFAULT true,
  target_user_percentage INTEGER DEFAULT 100,
  ab_test_variant VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  created_by VARCHAR(100),
  
  INDEX idx_journey_steps_active (is_active, priority),
  INDEX idx_journey_steps_category (category),
  INDEX idx_journey_steps_key (step_key)
);

-- User journey state tracking
CREATE TABLE user_journey_state (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  step_key VARCHAR(64) NOT NULL,
  status ENUM('eligible', 'shown', 'accepted', 'dismissed', 'expired', 'suppressed') NOT NULL,
  first_shown_at TIMESTAMP,
  last_shown_at TIMESTAMP,
  accepted_at TIMESTAMP,
  dismissed_at TIMESTAMP,
  dismissal_count INTEGER DEFAULT 0,
  shown_contexts JSONB DEFAULT '[]',
  dismissal_reasons JSONB DEFAULT '[]',
  assigned_variant VARCHAR(50),
  time_to_decision_ms INTEGER,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  
  UNIQUE(user_id, step_key),
  INDEX idx_user_journey_user (user_id),
  INDEX idx_user_journey_status (status),
  INDEX idx_user_journey_shown (last_shown_at)
);

-- High-volume analytics events
CREATE TABLE journey_analytics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  step_key VARCHAR(64) NOT NULL,
  event_type ENUM('shown', 'accepted', 'dismissed', 'eligible', 'suppressed') NOT NULL,
  user_context JSONB,
  app_context JSONB,
  session_id VARCHAR(100),
  triggered_by VARCHAR(100),
  referrer_screen VARCHAR(100),
  event_timestamp TIMESTAMP DEFAULT NOW(),
  
  INDEX idx_journey_analytics_user (user_id),
  INDEX idx_journey_analytics_step (step_key),
  INDEX idx_journey_analytics_event (event_type),
  INDEX idx_journey_analytics_time (event_timestamp)
);

-- ===================================================================
-- SENTIMENT ANALYSIS TABLES
-- ===================================================================

-- Message sentiment data
CREATE TABLE message_sentiment (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  message_id UUID NOT NULL REFERENCES messages(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  sentiment VARCHAR(20) NOT NULL, -- 'positive', 'negative', 'neutral', 'mixed'
  confidence DECIMAL(3,2) NOT NULL, -- 0.00 to 1.00
  emotional_state JSONB, -- {primary: 'excited', all: ['excited', 'grateful'], intensity: 0.8}
  personality_traits JSONB, -- {creative: 0.7, social: 0.9, professional: 0.2}
  analyzed_at TIMESTAMP DEFAULT NOW(),
  
  INDEX idx_sentiment_user (user_id),
  INDEX idx_sentiment_message (message_id),
  INDEX idx_sentiment_type (sentiment),
  INDEX idx_sentiment_time (analyzed_at)
);

-- User personality profile (evolving over time)
CREATE TABLE user_personality_profile (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  creative_score DECIMAL(3,2) DEFAULT 0.5,
  social_score DECIMAL(3,2) DEFAULT 0.5,
  professional_score DECIMAL(3,2) DEFAULT 0.5,
  emotional_score DECIMAL(3,2) DEFAULT 0.5,
  humorous_score DECIMAL(3,2) DEFAULT 0.5,
  analytical_score DECIMAL(3,2) DEFAULT 0.5,
  dominant_trait VARCHAR(50),
  confidence_level DECIMAL(3,2) DEFAULT 0.3,
  last_updated TIMESTAMP DEFAULT NOW(),
  
  UNIQUE(user_id),
  INDEX idx_personality_user (user_id),
  INDEX idx_personality_trait (dominant_trait)
);

-- ===================================================================
-- CHURN PREDICTION TABLES
-- ===================================================================

-- Churn risk assessments
CREATE TABLE churn_risk_assessments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  risk_score INTEGER NOT NULL, -- 0-100
  risk_level VARCHAR(20) NOT NULL, -- 'minimal', 'low', 'medium', 'high', 'critical'
  risk_factors JSONB NOT NULL, -- Array of risk factor strings
  prediction_timeframe VARCHAR(50), -- 'within_48_hours', 'within_week', etc.
  prediction_probability INTEGER, -- 0-100
  prediction_confidence DECIMAL(3,2),
  assessed_at TIMESTAMP DEFAULT NOW(),
  
  INDEX idx_churn_risk_user (user_id),
  INDEX idx_churn_risk_level (risk_level),
  INDEX idx_churn_risk_time (assessed_at)
);

-- Churn prevention actions
CREATE TABLE churn_prevention_actions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  action_type VARCHAR(100) NOT NULL,
  priority VARCHAR(20) NOT NULL,
  channel VARCHAR(50) NOT NULL, -- 'push_notification', 'email', 'in_app', 'sms'
  content TEXT NOT NULL,
  scheduled_for TIMESTAMP NOT NULL,
  executed_at TIMESTAMP,
  result JSONB, -- Success/failure details
  effectiveness_score DECIMAL(3,2), -- Measured later
  created_at TIMESTAMP DEFAULT NOW(),
  
  INDEX idx_prevention_user (user_id),
  INDEX idx_prevention_scheduled (scheduled_for),
  INDEX idx_prevention_executed (executed_at)
);

-- ===================================================================
-- FEATURE INTELLIGENCE TABLES
-- ===================================================================

-- Cross-feature usage patterns
CREATE TABLE feature_usage_patterns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  feature_name VARCHAR(100) NOT NULL,
  usage_context JSONB NOT NULL,
  related_features JSONB, -- Features used before/after
  outcome JSONB, -- Success metrics
  recorded_at TIMESTAMP DEFAULT NOW(),
  
  INDEX idx_feature_usage_user (user_id),
  INDEX idx_feature_usage_feature (feature_name),
  INDEX idx_feature_usage_time (recorded_at)
);

-- User feature affinity scores
CREATE TABLE user_feature_affinity (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  feature_name VARCHAR(100) NOT NULL,
  affinity_score DECIMAL(3,2) NOT NULL, -- 0.00 to 1.00
  usage_frequency INTEGER DEFAULT 0,
  last_used TIMESTAMP,
  predicted_usage DECIMAL(3,2), -- ML prediction
  updated_at TIMESTAMP DEFAULT NOW(),
  
  UNIQUE(user_id, feature_name),
  INDEX idx_affinity_user (user_id),
  INDEX idx_affinity_feature (feature_name),
  INDEX idx_affinity_score (affinity_score)
);
```

### **Existing Table Modifications**

```sql
-- Add fields to existing users table
ALTER TABLE users ADD COLUMN journey_preferences JSONB DEFAULT '{}';
ALTER TABLE users ADD COLUMN last_journey_shown TIMESTAMP;
ALTER TABLE users ADD COLUMN journey_tolerance DECIMAL(3,2) DEFAULT 0.5;

-- Add sentiment tracking to messages table  
ALTER TABLE messages ADD COLUMN sentiment_analyzed BOOLEAN DEFAULT FALSE;
ALTER TABLE messages ADD COLUMN sentiment_score DECIMAL(3,2);
```

---

## 🔧 **Backend Implementation**

### **1. Journey Orchestrator Service**

```javascript
// api/services/JourneyOrchestratorService.js
class JourneyOrchestratorService {
  
  async evaluateUserJourneys(userId, context = {}) {
    // Get user profile and current state
    const user = await this.getUserProfile(userId);
    const currentContext = await this.getContextualIntelligence(userId);
    
    // Find applicable journey steps
    const eligibleSteps = await this.findEligibleJourneySteps(user, currentContext);
    
    // Apply intelligent filtering
    const filteredSteps = await this.applyIntelligentFiltering(
      eligibleSteps, 
      user, 
      currentContext
    );
    
    // Select optimal step
    const optimalStep = await this.selectOptimalStep(filteredSteps, user);
    
    if (optimalStep) {
      // Track analytics
      await this.trackJourneyEvent(userId, optimalStep, 'eligible');
      
      // Adapt step for current context
      const adaptedStep = await this.adaptStepForContext(optimalStep, currentContext);
      
      return adaptedStep;
    }
    
    return null;
  }
  
  async findEligibleJourneySteps(user, context) {
    const steps = await models.JourneyStep.findAll({
      where: {
        isActive: true,
        [Op.or]: [
          { expiresAfterDays: null },
          { 
            createdAt: { 
              [Op.gte]: new Date(Date.now() - (expiresAfterDays * 24 * 60 * 60 * 1000))
            }
          }
        ]
      },
      order: [['priority', 'ASC']]
    });
    
    const eligible = [];
    
    for (const step of steps) {
      // Check if user has already completed/dismissed
      const userState = await models.UserJourneyState.findOne({
        where: { userId: user.id, stepKey: step.stepKey }
      });
      
      if (userState && 
          (userState.status === 'accepted' || 
           userState.dismissalCount >= step.maxDismissals)) {
        continue;
      }
      
      // Check cooldown
      if (userState && userState.lastShownAt) {
        const hoursSinceShown = (Date.now() - userState.lastShownAt) / (1000 * 60 * 60);
        if (hoursSinceShown < step.cooldownHours) {
          continue;
        }
      }
      
      // Evaluate trigger condition
      if (await this.evaluateTriggerCondition(step.triggerConfig, user, context)) {
        eligible.push(step);
      }
    }
    
    return eligible;
  }
  
  async applyIntelligentFiltering(steps, user, context) {
    const filtered = [];
    
    for (const step of steps) {
      // Sentiment-based filtering
      if (context.sentiment && context.sentiment.sentiment === 'negative' && 
          step.category === 'conversion') {
        continue; // Don't show conversion journeys when user is negative
      }
      
      // Stress-based filtering
      if (context.stressLevel === 'high' && step.complexity === 'high') {
        continue; // Don't show complex journeys when user is stressed
      }
      
      // Churn risk filtering
      if (context.churnRisk && context.churnRisk.level === 'critical' && 
          step.category !== 'retention') {
        continue; // Only retention journeys for critical churn risk
      }
      
      // Timing-based filtering
      if (!await this.isOptimalTiming(user, step, context)) {
        continue;
      }
      
      filtered.push(step);
    }
    
    return filtered;
  }
  
  async evaluateTriggerCondition(triggerConfig, user, context) {
    switch (triggerConfig.type) {
      case 'user_field_check':
        return this.evaluateUserFieldCheck(triggerConfig, user);
        
      case 'count_threshold':
        return await this.evaluateCountThreshold(triggerConfig, user);
        
      case 'time_since':
        return this.evaluateTimeSince(triggerConfig, user);
        
      case 'composite':
        return await this.evaluateComposite(triggerConfig, user, context);
        
      case 'sentiment_based':
        return this.evaluateSentiment(triggerConfig, context);
        
      case 'behavioral_pattern':
        return await this.evaluateBehavioralPattern(triggerConfig, user);
        
      default:
        return false;
    }
  }
}
```

### **2. Sentiment Analysis Service**

```javascript
// api/services/SentimentAnalysisService.js
class SentimentAnalysisService {
  
  async analyzeMessage(messageText, userId, messageId) {
    try {
      const analysis = {
        basicSentiment: this.calculateBasicSentiment(messageText),
        emotionalState: this.detectEmotionalState(messageText),
        personalityTraits: this.extractPersonalityTraits(messageText),
        intentAnalysis: this.analyzeIntent(messageText)
      };
      
      // Store in database
      await models.MessageSentiment.create({
        messageId,
        userId,
        sentiment: analysis.basicSentiment.sentiment,
        confidence: analysis.basicSentiment.confidence,
        emotionalState: analysis.emotionalState,
        personalityTraits: analysis.personalityTraits,
        analyzedAt: new Date()
      });
      
      // Update user personality profile
      await this.updateUserPersonalityProfile(userId, analysis.personalityTraits);
      
      // Trigger journey adjustments
      await this.adjustUserJourneysBasedOnSentiment(userId, analysis);
      
      return analysis;
      
    } catch (error) {
      Logger.error('Sentiment analysis failed:', error);
      return null;
    }
  }
  
  calculateBasicSentiment(text) {
    const positiveWords = [
      'love', 'amazing', 'great', 'wonderful', 'happy', 'excited', 'fantastic',
      'awesome', 'perfect', 'brilliant', 'excellent', 'delighted', 'thrilled'
    ];
    
    const negativeWords = [
      'hate', 'terrible', 'awful', 'sad', 'angry', 'frustrated', 'annoyed',
      'disappointed', 'stressed', 'overwhelmed', 'anxious', 'worried', 'upset'
    ];
    
    const words = text.toLowerCase().split(/\s+/);
    let positiveScore = 0;
    let negativeScore = 0;
    
    words.forEach(word => {
      if (positiveWords.includes(word)) positiveScore++;
      if (negativeWords.includes(word)) negativeScore++;
    });
    
    const totalEmotionalWords = positiveScore + negativeScore;
    
    if (totalEmotionalWords === 0) {
      return { sentiment: 'neutral', confidence: 0.3 };
    }
    
    const positiveRatio = positiveScore / totalEmotionalWords;
    const negativeRatio = negativeScore / totalEmotionalWords;
    
    if (positiveRatio > 0.6) {
      return { sentiment: 'positive', confidence: positiveRatio };
    } else if (negativeRatio > 0.6) {
      return { sentiment: 'negative', confidence: negativeRatio };
    } else {
      return { sentiment: 'mixed', confidence: 0.5 };
    }
  }
  
  async updateUserPersonalityProfile(userId, newTraits) {
    let profile = await models.UserPersonalityProfile.findOne({
      where: { userId }
    });
    
    if (!profile) {
      profile = await models.UserPersonalityProfile.create({
        userId,
        ...newTraits,
        lastUpdated: new Date()
      });
    } else {
      // Weighted average with existing profile (new data gets 20% weight)
      const weight = 0.2;
      profile.creativeScore = (profile.creativeScore * (1 - weight)) + (newTraits.creative * weight);
      profile.socialScore = (profile.socialScore * (1 - weight)) + (newTraits.social * weight);
      profile.professionalScore = (profile.professionalScore * (1 - weight)) + (newTraits.professional * weight);
      profile.emotionalScore = (profile.emotionalScore * (1 - weight)) + (newTraits.emotional * weight);
      profile.humorousScore = (profile.humorousScore * (1 - weight)) + (newTraits.humorous * weight);
      profile.analyticalScore = (profile.analyticalScore * (1 - weight)) + (newTraits.analytical * weight);
      
      // Update dominant trait
      const scores = {
        creative: profile.creativeScore,
        social: profile.socialScore,
        professional: profile.professionalScore,
        emotional: profile.emotionalScore,
        humorous: profile.humorousScore,
        analytical: profile.analyticalScore
      };
      
      profile.dominantTrait = Object.keys(scores).reduce((a, b) => 
        scores[a] > scores[b] ? a : b
      );
      
      profile.lastUpdated = new Date();
      await profile.save();
    }
    
    return profile;
  }
}
```

### **3. Churn Prediction Service**

```javascript
// api/services/ChurnPredictionService.js
class ChurnPredictionService {
  
  async calculateChurnRisk(userId) {
    const factors = await this.getChurnFactors(userId);
    let riskScore = 0;
    const riskFactors = [];
    
    // Engagement factors (strongest predictors)
    if (factors.daysSinceLastActive >= 7) {
      riskScore += 40;
      riskFactors.push('inactive_7_days');
    } else if (factors.daysSinceLastActive >= 3) {
      riskScore += 20;
      riskFactors.push('inactive_3_days');
    }
    
    // Onboarding completion
    if (factors.totalMessageCount === 0) {
      riskScore += 50;
      riskFactors.push('never_activated');
    }
    
    if (!factors.hasCreatedCharacter) {
      riskScore += 35;
      riskFactors.push('no_character_created');
    }
    
    if (!factors.hasAddedRecipient) {
      riskScore += 30;
      riskFactors.push('no_recipient_added');
    }
    
    // Positive factors (reduce risk)
    if (factors.hasActiveSubscription) {
      riskScore -= 40;
      riskFactors.push('has_subscription');
    }
    
    if (factors.totalMessageCount > 20) {
      riskScore -= 25;
      riskFactors.push('high_message_count');
    }
    
    const finalRiskScore = Math.max(0, Math.min(100, riskScore));
    const riskLevel = this.categorizeRiskLevel(finalRiskScore);
    
    // Store assessment
    await models.ChurnRiskAssessment.create({
      userId,
      riskScore: finalRiskScore,
      riskLevel,
      riskFactors,
      predictionTimeframe: this.getPredictionTimeframe(finalRiskScore),
      predictionProbability: Math.min(95, finalRiskScore),
      predictionConfidence: this.calculatePredictionConfidence(factors),
      assessedAt: new Date()
    });
    
    // If high risk, generate prevention actions
    if (finalRiskScore >= 60) {
      await this.generatePreventionActions(userId, finalRiskScore, riskFactors);
    }
    
    return {
      userId,
      riskScore: finalRiskScore,
      riskLevel,
      riskFactors,
      needsIntervention: finalRiskScore >= 60
    };
  }
  
  async generatePreventionActions(userId, riskScore, riskFactors) {
    const actions = [];
    
    if (riskScore >= 80) {
      actions.push({
        type: 'immediate_personal_outreach',
        priority: 'critical',
        channel: 'email',
        content: 'Personal note from founder asking for feedback',
        scheduledFor: new Date()
      });
    }
    
    if (riskFactors.includes('never_activated')) {
      actions.push({
        type: 'activation_assistance',
        priority: 'high',
        channel: 'push_notification',
        content: 'Let us help you get started with your first message',
        scheduledFor: new Date(Date.now() + 4 * 60 * 60 * 1000) // 4 hours
      });
    }
    
    if (riskFactors.includes('declining_usage')) {
      actions.push({
        type: 'usage_motivation',
        priority: 'medium',
        channel: 'in_app',
        content: 'Your messages make a difference - here\'s why',
        scheduledFor: new Date(Date.now() + 12 * 60 * 60 * 1000) // 12 hours
      });
    }
    
    // Store prevention actions
    for (const action of actions) {
      await models.ChurnPreventionAction.create({
        userId,
        actionType: action.type,
        priority: action.priority,
        channel: action.channel,
        content: action.content,
        scheduledFor: action.scheduledFor
      });
    }
    
    return actions;
  }
}
```

---

## 📱 **Frontend Implementation**

### **1. Enhanced Journey Hook**

```javascript
// src/hooks/useIntelligentJourney.js
import { useState, useEffect, useCallback } from 'react';
import { journeyApi } from '../apis/journey';
import { trackJourneyEvent } from '../utils/analytics';

const useIntelligentJourney = () => {
  const [currentJourney, setCurrentJourney] = useState(null);
  const [showJourneyModal, setShowJourneyModal] = useState(false);
  const [journeyContext, setJourneyContext] = useState({});
  
  const checkForJourney = useCallback(async (context = {}) => {
    try {
      const journey = await journeyApi.evaluateJourneys(context);
      
      if (journey) {
        // Track that journey was eligible
        await trackJourneyEvent('journey_eligible', {
          journeyKey: journey.stepKey,
          context,
          riskScore: context.churnRisk?.riskScore,
          sentiment: context.sentiment?.sentiment
        });
        
        setCurrentJourney(journey);
        setJourneyContext(context);
        setShowJourneyModal(true);
      }
    } catch (error) {
      console.error('Journey evaluation failed:', error);
    }
  }, []);
  
  const handleJourneyAction = useCallback(async (action) => {
    if (!currentJourney) return;
    
    const startTime = Date.now();
    
    try {
      if (action === 'accept') {
        // Track acceptance
        await trackJourneyEvent('journey_accepted', {
          journeyKey: currentJourney.stepKey,
          timeToDecision: Date.now() - startTime,
          context: journeyContext
        });
        
        // Record in backend
        await journeyApi.recordJourneyAction(currentJourney.stepKey, 'accepted');
        
        // Navigate if specified
        if (currentJourney.destinationScreen) {
          navigation.navigate(currentJourney.destinationScreen, currentJourney.destinationParams);
        }
        
      } else if (action === 'dismiss') {
        // Track dismissal
        await trackJourneyEvent('journey_dismissed', {
          journeyKey: currentJourney.stepKey,
          timeToDecision: Date.now() - startTime,
          context: journeyContext
        });
        
        // Record in backend
        await journeyApi.recordJourneyAction(currentJourney.stepKey, 'dismissed');
      }
      
    } finally {
      setShowJourneyModal(false);
      setCurrentJourney(null);
      setJourneyContext({});
    }
  }, [currentJourney, journeyContext]);
  
  // Auto-check for journeys on key events
  const onMessageSent = useCallback(async (messageData) => {
    await checkForJourney({
      trigger: 'message_sent',
      messageData,
      timestamp: new Date()
    });
  }, [checkForJourney]);
  
  const onCharacterCreated = useCallback(async (characterData) => {
    await checkForJourney({
      trigger: 'character_created',
      characterData,
      timestamp: new Date()
    });
  }, [checkForJourney]);
  
  const onRecipientAdded = useCallback(async (recipientData) => {
    await checkForJourney({
      trigger: 'recipient_added',
      recipientData,
      timestamp: new Date()
    });
  }, [checkForJourney]);
  
  const onAppOpen = useCallback(async () => {
    await checkForJourney({
      trigger: 'app_open',
      timestamp: new Date()
    });
  }, [checkForJourney]);
  
  return {
    currentJourney,
    showJourneyModal,
    handleJourneyAction,
    checkForJourney,
    
    // Event handlers
    onMessageSent,
    onCharacterCreated,
    onRecipientAdded,
    onAppOpen
  };
};

export default useIntelligentJourney;
```

### **2. Enhanced Choice Modal**

```javascript
// src/components/modals/IntelligentJourneyModal.js
import React from 'react';
import { Modal } from 'native-base';
import { colors, spacing, textStyles } from '../../theme';

const IntelligentJourneyModal = ({ 
  isVisible, 
  journey, 
  onAction,
  context = {} 
}) => {
  if (!journey) return null;
  
  // Adapt modal styling based on context
  const getModalStyle = () => {
    if (context.sentiment?.sentiment === 'negative') {
      return { backgroundColor: colors.surface, borderColor: colors.supportive };
    }
    if (context.churnRisk?.level === 'critical') {
      return { backgroundColor: colors.warning.light, borderColor: colors.warning.main };
    }
    return { backgroundColor: colors.surface };
  };
  
  const getActionButtonStyle = (isPrimary) => {
    const baseStyle = {
      paddingVertical: spacing.md,
      paddingHorizontal: spacing.lg,
      borderRadius: spacing.sm,
      marginVertical: spacing.xs
    };
    
    if (isPrimary) {
      return {
        ...baseStyle,
        backgroundColor: colors.primary,
      };
    } else {
      return {
        ...baseStyle,
        backgroundColor: colors.surface,
        borderWidth: 1,
        borderColor: colors.border
      };
    }
  };
  
  return (
    <Modal isOpen={isVisible} onClose={() => onAction('dismiss')}>
      <Modal.Content style={getModalStyle()}>
        <Modal.Header>
          <Text style={textStyles.modalTitle}>
            {journey.title}
          </Text>
        </Modal.Header>
        
        <Modal.Body>
          <Text style={textStyles.body}>
            {journey.message}
          </Text>
          
          {/* Show context-aware additional content */}
          {context.sentiment?.sentiment === 'positive' && (
            <Text style={[textStyles.caption, { color: colors.success, marginTop: spacing.sm }]}>
              ✨ Perfect timing - you seem to be in a great mood!
            </Text>
          )}
          
          {context.churnRisk?.level === 'high' && (
            <Text style={[textStyles.caption, { color: colors.warning.main, marginTop: spacing.sm }]}>
              💡 This feature might make LittleTalks even more valuable for you
            </Text>
          )}
        </Modal.Body>
        
        <Modal.Footer>
          <VStack space={spacing.sm} width="100%">
            <TouchableOpacity 
              style={getActionButtonStyle(true)}
              onPress={() => onAction('accept')}
            >
              <Text style={[textStyles.buttonPrimary, { textAlign: 'center' }]}>
                {journey.primaryAction}
              </Text>
            </TouchableOpacity>
            
            <TouchableOpacity 
              style={getActionButtonStyle(false)}
              onPress={() => onAction('dismiss')}
            >
              <Text style={[textStyles.buttonSecondary, { textAlign: 'center' }]}>
                {journey.secondaryAction}
              </Text>
            </TouchableOpacity>
          </VStack>
        </Modal.Footer>
      </Modal.Content>
    </Modal>
  );
};

export default IntelligentJourneyModal;
```

### **3. Integration Points**

```javascript
// src/screens/Chat/index.js - Enhanced with intelligent journeys
import useIntelligentJourney from '../../hooks/useIntelligentJourney';
import IntelligentJourneyModal from '../../components/modals/IntelligentJourneyModal';

const ChatScreen = () => {
  const { 
    currentJourney, 
    showJourneyModal, 
    handleJourneyAction,
    onMessageSent 
  } = useIntelligentJourney();
  
  const handleSendMessage = async (messageData) => {
    // ... existing message sending logic ...
    
    // Trigger intelligent journey check
    await onMessageSent({
      messageText: messageData.content,
      recipientId: messageData.recipientId,
      characterId: messageData.characterId,
      sentiment: messageData.analyzedSentiment
    });
  };
  
  return (
    <View>
      {/* Existing chat UI */}
      
      {/* Intelligent Journey Modal */}
      <IntelligentJourneyModal
        isVisible={showJourneyModal}
        journey={currentJourney}
        onAction={handleJourneyAction}
        context={{
          sentiment: currentJourney?.context?.sentiment,
          churnRisk: currentJourney?.context?.churnRisk
        }}
      />
    </View>
  );
};
```

---

## 📊 **Analytics & Monitoring**

### **Key Metrics to Track**

```javascript
// src/utils/journeyAnalytics.js
class JourneyAnalyticsService {
  
  // Core journey metrics
  async trackJourneyEvent(eventType, data) {
    const event = {
      event: `journey_${eventType}`,
      properties: {
        journey_key: data.journeyKey,
        user_id: data.userId,
        timestamp: new Date().toISOString(),
        
        // Context data
        sentiment: data.sentiment,
        churn_risk_score: data.churnRisk?.riskScore,
        personality_profile: data.personalityProfile,
        
        // Behavioral data
        time_to_decision_ms: data.timeToDecision,
        session_duration: data.sessionDuration,
        app_version: data.appVersion,
        
        // Journey performance
        journey_priority: data.journeyPriority,
        journey_category: data.journeyCategory,
        ab_test_variant: data.abTestVariant
      }
    };
    
    // Send to multiple analytics providers
    await Promise.all([
      this.sendToGA4(event),
      this.sendToMixpanel(event),
      this.storeInDatabase(event)
    ]);
  }
  
  // Sentiment analysis metrics
  async trackSentimentAnalysis(userId, analysis) {
    await this.trackJourneyEvent('sentiment_analyzed', {
      userId,
      sentiment: analysis.basicSentiment.sentiment,
      confidence: analysis.basicSentiment.confidence,
      emotional_state: analysis.emotionalState.primaryEmotion,
      personality_traits: analysis.personalityInsights.dominantTrait
    });
  }
  
  // Churn prediction metrics
  async trackChurnPrediction(userId, assessment) {
    await this.trackJourneyEvent('churn_risk_assessed', {
      userId,
      risk_score: assessment.riskScore,
      risk_level: assessment.riskLevel,
      risk_factors: assessment.riskFactors,
      prediction_timeframe: assessment.prediction.timeframe
    });
  }
}
```

### **Dashboard Metrics**

```sql
-- Journey performance queries for admin dashboard

-- Journey acceptance rates by category
SELECT 
  js.category,
  COUNT(CASE WHEN ja.event_type = 'shown' THEN 1 END) as shown_count,
  COUNT(CASE WHEN ja.event_type = 'accepted' THEN 1 END) as accepted_count,
  ROUND(
    COUNT(CASE WHEN ja.event_type = 'accepted' THEN 1 END)::decimal / 
    COUNT(CASE WHEN ja.event_type = 'shown' THEN 1 END) * 100, 
    2
  ) as acceptance_rate
FROM journey_steps js
LEFT JOIN journey_analytics ja ON js.step_key = ja.step_key
WHERE ja.event_timestamp >= NOW() - INTERVAL '30 days'
GROUP BY js.category
ORDER BY acceptance_rate DESC;

-- Sentiment impact on journey success
SELECT 
  JSON_EXTRACT(ja.user_context, '$.sentiment') as sentiment,
  COUNT(CASE WHEN ja.event_type = 'shown' THEN 1 END) as shown_count,
  COUNT(CASE WHEN ja.event_type = 'accepted' THEN 1 END) as accepted_count,
  ROUND(
    COUNT(CASE WHEN ja.event_type = 'accepted' THEN 1 END)::decimal / 
    COUNT(CASE WHEN ja.event_type = 'shown' THEN 1 END) * 100, 
    2
  ) as acceptance_rate
FROM journey_analytics ja
WHERE ja.event_timestamp >= NOW() - INTERVAL '30 days'
  AND JSON_EXTRACT(ja.user_context, '$.sentiment') IS NOT NULL
GROUP BY JSON_EXTRACT(ja.user_context, '$.sentiment')
ORDER BY acceptance_rate DESC;

-- Churn prevention effectiveness
SELECT 
  cpa.action_type,
  COUNT(*) as actions_taken,
  COUNT(CASE WHEN cpa.effectiveness_score > 0.5 THEN 1 END) as successful_actions,
  ROUND(
    COUNT(CASE WHEN cpa.effectiveness_score > 0.5 THEN 1 END)::decimal / 
    COUNT(*) * 100, 
    2
  ) as success_rate
FROM churn_prevention_actions cpa
WHERE cpa.executed_at >= NOW() - INTERVAL '30 days'
  AND cpa.effectiveness_score IS NOT NULL
GROUP BY cpa.action_type
ORDER BY success_rate DESC;
```

---

## 🧪 **Testing Strategy**

### **1. A/B Testing Framework**

```javascript
// src/utils/abTesting.js
class ABTestingService {
  
  async getJourneyVariant(userId, journeyKey) {
    const journey = await models.JourneyStep.findOne({
      where: { stepKey: journeyKey }
    });
    
    if (!journey.abTestVariant) {
      return journey; // No A/B test
    }
    
    // Determine user variant based on user ID hash
    const userHash = this.hashUserId(userId);
    const variant = userHash % 100 < journey.targetUserPercentage ? 'treatment' : 'control';
    
    if (variant === 'control') {
      return null; // Don't show journey to control group
    }
    
    // Track variant assignment
    await models.UserJourneyState.upsert({
      userId,
      stepKey: journeyKey,
      assignedVariant: variant
    });
    
    return journey;
  }
  
  async analyzeABTestResults(journeyKey, days = 30) {
    const results = await models.sequelize.query(`
      SELECT 
        ujs.assigned_variant,
        COUNT(CASE WHEN ja.event_type = 'shown' THEN 1 END) as shown_count,
        COUNT(CASE WHEN ja.event_type = 'accepted' THEN 1 END) as accepted_count,
        COUNT(CASE WHEN ja.event_type = 'dismissed' THEN 1 END) as dismissed_count
      FROM user_journey_state ujs
      LEFT JOIN journey_analytics ja ON ujs.user_id = ja.user_id 
        AND ujs.step_key = ja.step_key
      WHERE ujs.step_key = :journeyKey
        AND ja.event_timestamp >= NOW() - INTERVAL :days DAY
      GROUP BY ujs.assigned_variant
    `, {
      replacements: { journeyKey, days },
      type: models.sequelize.QueryTypes.SELECT
    });
    
    return this.calculateStatisticalSignificance(results);
  }
}
```

### **2. Integration Testing**

```javascript
// __tests__/integration/journeySystem.test.js
describe('Intelligent Journey System Integration', () => {
  
  test('should show character creation journey for new user', async () => {
    const newUser = await createTestUser({ hasCreatedCharacter: false });
    
    const journey = await journeyService.evaluateUserJourneys(newUser.id, {
      trigger: 'app_open'
    });
    
    expect(journey).toBeTruthy();
    expect(journey.stepKey).toBe('create_first_character');
  });
  
  test('should not show conversion journeys when user sentiment is negative', async () => {
    const user = await createTestUser();
    await createTestSentiment(user.id, { sentiment: 'negative', confidence: 0.8 });
    
    const journey = await journeyService.evaluateUserJourneys(user.id, {
      trigger: 'message_sent'
    });
    
    // Should not get conversion-category journeys
    expect(journey?.category).not.toBe('conversion');
  });
  
  test('should trigger churn prevention for high-risk users', async () => {
    const user = await createTestUser({
      lastActiveAt: new Date(Date.now() - 8 * 24 * 60 * 60 * 1000) // 8 days ago
    });
    
    const churnRisk = await churnService.calculateChurnRisk(user.id);
    expect(churnRisk.riskLevel).toBe('high');
    
    const preventionActions = await models.ChurnPreventionAction.findAll({
      where: { userId: user.id }
    });
    
    expect(preventionActions.length).toBeGreaterThan(0);
  });
});
```

---

## 🚀 **Deployment Strategy**

### **Phase 1: Foundation (Weeks 1-4)**
1. **Database migrations** - Create all new tables
2. **Basic journey orchestrator** - Core journey evaluation logic
3. **Simple sentiment analysis** - Keyword-based sentiment detection
4. **Frontend integration** - Enhanced journey hook and modal

### **Phase 2: Intelligence (Weeks 5-8)**
1. **Cross-feature learning** - Pattern recognition between features
2. **Contextual awareness** - Time-based and behavioral filtering
3. **Personality profiling** - Build user personality models
4. **A/B testing framework** - Test journey effectiveness

### **Phase 3: Prediction (Weeks 9-12)**
1. **Churn prediction** - Risk assessment and prevention
2. **Advanced sentiment analysis** - ML-powered emotion detection
3. **Seasonal campaigns** - Holiday and event-based journeys
4. **Real-time adaptation** - Dynamic journey modification

### **Phase 4: Optimization (Weeks 13-24)**
1. **Machine learning models** - Advanced prediction algorithms
2. **Advanced analytics** - Comprehensive performance tracking
3. **Auto-optimization** - Self-improving journey selection
4. **Scale optimization** - Performance and cost optimization

---

## 📋 **Implementation Checklist**

### **Backend Development**
- [ ] Create database migrations for all new tables
- [ ] Implement JourneyOrchestratorService
- [ ] Build SentimentAnalysisService  
- [ ] Create ChurnPredictionService
- [ ] Add API endpoints for journey evaluation
- [ ] Implement analytics tracking
- [ ] Build A/B testing framework
- [ ] Create admin dashboard queries

### **Frontend Development**
- [ ] Create useIntelligentJourney hook
- [ ] Build IntelligentJourneyModal component
- [ ] Integrate with existing screens (Chat, Character, Recipient)
- [ ] Add analytics tracking to all interactions
- [ ] Implement journey state management
- [ ] Create journey testing utilities
- [ ] Add accessibility support

### **Infrastructure**
- [ ] Set up analytics data pipeline
- [ ] Configure monitoring and alerting
- [ ] Plan database scaling for analytics data
- [ ] Implement rate limiting for journey checks
- [ ] Set up A/B testing infrastructure
- [ ] Create backup and recovery procedures

### **Testing**
- [ ] Write unit tests for all services
- [ ] Create integration tests for journey flows
- [ ] Build performance tests for high-volume scenarios
- [ ] Implement A/B testing validation
- [ ] Create user acceptance testing scenarios
- [ ] Set up automated regression testing

### **Monitoring & Analytics**
- [ ] Set up journey performance dashboards
- [ ] Implement real-time alerting for system issues
- [ ] Create business metrics tracking
- [ ] Build A/B testing result analysis
- [ ] Set up user feedback collection
- [ ] Implement system health monitoring

---

## 🎯 **Success Criteria**

### **Technical Success**
- [ ] System processes 10,000+ journey evaluations per day
- [ ] 95%+ uptime for journey recommendation system
- [ ] <200ms response time for journey evaluation
- [ ] Zero data loss in analytics pipeline
- [ ] Successful A/B testing of 5+ journey variants

### **Business Success**
- [ ] 40% increase in user activation rate
- [ ] 35% improvement in 7-day retention
- [ ] 60% boost in feature discovery
- [ ] 25% increase in conversion rate
- [ ] 67% churn prevention success rate

### **User Experience Success**
- [ ] <5% negative feedback on journey prompts
- [ ] >70% journey acceptance rate
- [ ] >80% user satisfaction with personalization
- [ ] <1% support tickets related to journey system
- [ ] >90% accessibility compliance

---

This implementation guide provides the complete technical roadmap for building LittleTalks' Intelligent Journey System. The modular architecture allows for phased implementation while the comprehensive testing strategy ensures reliability and performance at scale.