# Learning Analytics Framework

## Overview
Advanced learning analytics and intelligence specifications for data-driven insights and personalized learning.

## Description
Comprehensive learning analytics framework including xAPI implementation, predictive analytics capabilities, learning outcome correlation analysis, and real-time performance dashboards for educational insights.

## Purpose
This specification defines the advanced learning analytics framework for the MLC-LMS platform, enabling data-driven insights that go beyond basic reporting to provide predictive analytics, personalized learning recommendations, and research-quality educational data analysis. The framework transforms raw event data into actionable intelligence for educators, administrators, researchers, and the platform itself, supporting continuous improvement of learning outcomes while maintaining appropriate privacy and ethical standards.

## Scope
**Included**
- xAPI (Experience API/Tin Can API) implementation and statement generation
- Predictive analytics models for student success and intervention triggers
- Learning outcome correlation analysis and pattern recognition
- Real-time performance dashboards with actionable insights
- Adaptive learning recommendations based on analytics
- Research and development analytics capabilities
- Cross-population learning insights (anonymized)
- Machine learning pipelines for personalization
- A/B testing framework for educational interventions
- Learning path optimization algorithms
- Retention and engagement prediction models

**Excluded**
- Basic event tracking infrastructure (covered in [event-model.md](./event-model.md))
- Standard KPI calculations and reporting (see [teacher-reports-kpis.md](./teacher-reports-kpis.md) and [admin-reports-kpis.md](./admin-reports-kpis.md))
- Operational dashboards for system administrators (see [sysadmin-ops-dashboards.md](./sysadmin-ops-dashboards.md))
- Specific dashboard UI implementations (covered in UX experience documents)
- Third-party analytics tool configurations (implementation-specific)

## Data model (if applicable)

### xAPI Statement Schema

The platform implements Experience API (xAPI) for standardized learning record statements that enable interoperability with external learning systems and research platforms.

#### Core xAPI Statement Structure
```json
{
  "id": "12345678-1234-5678-1234-567812345678",
  "actor": {
    "objectType": "Agent",
    "account": {
      "homePage": "https://musiclearningcommunity.com",
      "name": "usr_12345"
    }
  },
  "verb": {
    "id": "http://adlnet.gov/expapi/verbs/completed",
    "display": { "en-US": "completed" }
  },
  "object": {
    "objectType": "Activity",
    "id": "https://musiclearningcommunity.com/games/G-01542/quiz",
    "definition": {
      "name": { "en-US": "Staff Birds Quiz" },
      "description": { "en-US": "Quiz stage for treble staff note recognition" },
      "type": "http://adlnet.gov/expapi/activities/assessment",
      "extensions": {
        "https://musiclearningcommunity.com/xapi/game-id": "G-01542",
        "https://musiclearningcommunity.com/xapi/stage-type": "quiz",
        "https://musiclearningcommunity.com/xapi/concept-tags": ["guide_notes", "treble_staff"]
      }
    }
  },
  "result": {
    "score": {
      "scaled": 0.87,
      "raw": 87,
      "min": 0,
      "max": 100
    },
    "success": true,
    "completion": true,
    "duration": "PT2M5S",
    "extensions": {
      "https://musiclearningcommunity.com/xapi/attempts": 2,
      "https://musiclearningcommunity.com/xapi/mastery-achieved": true,
      "https://musiclearningcommunity.com/xapi/input-mode": "midi"
    }
  },
  "context": {
    "contextActivities": {
      "parent": [
        {
          "id": "https://musiclearningcommunity.com/games/G-01542",
          "objectType": "Activity"
        }
      ],
      "grouping": [
        {
          "id": "https://musiclearningcommunity.com/sequences/seq_456",
          "objectType": "Activity"
        },
        {
          "id": "https://musiclearningcommunity.com/assignments/asg_789",
          "objectType": "Activity"
        }
      ]
    },
    "extensions": {
      "https://musiclearningcommunity.com/xapi/session-id": "sess_67890abcdef",
      "https://musiclearningcommunity.com/xapi/assignment-id": "asg_789"
    }
  },
  "timestamp": "2025-01-27T14:30:45.123Z"
}
```

#### MLC-Specific xAPI Verbs
```javascript
// Custom verbs for music learning activities
const MLC_XAPI_VERBS = {
  mastered: "https://musiclearningcommunity.com/xapi/verbs/mastered",
  reviewed: "https://musiclearningcommunity.com/xapi/verbs/reviewed",
  practiced: "https://musiclearningcommunity.com/xapi/verbs/practiced",
  challenged: "https://musiclearningcommunity.com/xapi/verbs/challenged",
  explored: "https://musiclearningcommunity.com/xapi/verbs/explored"
}
```

### Predictive Analytics Models

#### Student Success Prediction
```json
{
  "model_id": "student_success_v2.1",
  "student_id": "usr_12345",
  "prediction_date": "2025-01-27T14:30:00Z",
  "success_probability": 0.87,
  "risk_level": "low|medium|high",
  "confidence": 0.94,
  "contributing_factors": [
    {
      "factor": "engagement_score",
      "weight": 0.35,
      "value": 0.89,
      "impact": "positive"
    },
    {
      "factor": "mastery_rate",
      "weight": 0.30,
      "value": 0.82,
      "impact": "positive"
    },
    {
      "factor": "session_consistency",
      "weight": 0.20,
      "value": 0.65,
      "impact": "neutral"
    },
    {
      "factor": "review_completion",
      "weight": 0.15,
      "value": 0.45,
      "impact": "negative"
    }
  ],
  "recommended_interventions": [
    {
      "intervention_type": "review_reminder",
      "priority": "medium",
      "expected_impact": 0.12,
      "description": "Student has incomplete review stages, affecting retention"
    }
  ]
}
```

#### Retention Risk Model
```json
{
  "model_id": "retention_risk_v1.5",
  "student_id": "usr_12345",
  "prediction_date": "2025-01-27T14:30:00Z",
  "churn_probability": 0.23,
  "risk_level": "low",
  "time_to_churn_days": 45,
  "key_indicators": {
    "days_since_last_activity": 7,
    "session_frequency_trend": "declining",
    "assignment_completion_rate": 0.78,
    "peer_comparison": "below_average"
  },
  "intervention_triggers": [
    {
      "trigger": "inactivity_7_days",
      "action": "send_engagement_reminder",
      "activated": true
    }
  ]
}
```

### Learning Outcome Correlation

#### Concept Correlation Matrix
```json
{
  "analysis_id": "correlation_2025_01",
  "analysis_date": "2025-01-27T00:00:00Z",
  "sample_size": 5847,
  "correlations": [
    {
      "concept_a": "guide_notes",
      "concept_b": "treble_staff",
      "correlation_coefficient": 0.87,
      "p_value": 0.001,
      "significance": "high",
      "interpretation": "Students who master guide notes show strong performance in treble staff reading"
    },
    {
      "concept_a": "rhythm_patterns",
      "concept_b": "sight_reading",
      "correlation_coefficient": 0.72,
      "p_value": 0.005,
      "significance": "medium",
      "interpretation": "Rhythm pattern mastery moderately predicts sight-reading success"
    }
  ],
  "optimal_learning_paths": [
    {
      "path_id": "path_pitch_visual_01",
      "concepts": ["guide_notes", "treble_staff", "ledger_lines", "grand_staff"],
      "average_time_to_mastery_days": 21,
      "success_rate": 0.89,
      "student_count": 1247
    }
  ]
}
```

#### Learning Velocity Analysis
```json
{
  "student_id": "usr_12345",
  "velocity_score": 1.2,
  "velocity_trend": "accelerating",
  "baseline_velocity": 1.0,
  "contributing_factors": {
    "time_to_mastery_improvement": 0.15,
    "attempts_per_mastery_decrease": 0.08,
    "session_frequency_increase": 0.05
  },
  "predicted_sequence_completion": {
    "sequence_id": "seq_456",
    "estimated_completion_date": "2025-03-15T00:00:00Z",
    "confidence": 0.82
  }
}
```

### Adaptive Learning Recommendations

#### Personalization Profile
```json
{
  "student_id": "usr_12345",
  "learning_profile": {
    "learning_style": "visual_kinesthetic",
    "optimal_session_duration_minutes": 18,
    "optimal_difficulty_progression": "gradual",
    "preferred_input_mode": "midi",
    "engagement_triggers": ["challenges", "leaderboards", "badges"],
    "distraction_indicators": ["rapid_clicking", "stage_abandonment"]
  },
  "performance_patterns": {
    "best_time_of_day": "afternoon",
    "optimal_game_sequence_length": 5,
    "concept_strengths": ["pitch_aural", "rhythm"],
    "concept_challenges": ["chords", "harmony"],
    "mastery_velocity": 1.2
  },
  "recommendations": [
    {
      "recommendation_type": "game_suggestion",
      "game_id": "G-02145",
      "reason": "Fills identified gap in chord recognition",
      "confidence": 0.85,
      "priority": "high"
    },
    {
      "recommendation_type": "difficulty_adjustment",
      "current_level": 3,
      "recommended_level": 4,
      "reason": "Student consistently exceeding target scores",
      "confidence": 0.91,
      "priority": "medium"
    }
  ]
}
```

### Research Analytics Models

#### Cohort Analysis Schema
```json
{
  "cohort_id": "cohort_2025_q1_piano_adventures",
  "cohort_definition": {
    "criteria": "Students using Piano Adventures aligned sequences",
    "start_date": "2025-01-01T00:00:00Z",
    "end_date": "2025-03-31T23:59:59Z",
    "sample_size": 1247
  },
  "aggregate_metrics": {
    "average_mastery_rate": 0.78,
    "average_time_to_concept_mastery_days": 3.5,
    "completion_rate": 0.84,
    "retention_rate_30d": 0.89
  },
  "statistical_analysis": {
    "mean": 78.4,
    "median": 82.0,
    "std_deviation": 12.3,
    "confidence_interval_95": [76.1, 80.7]
  },
  "comparison_cohorts": [
    {
      "cohort_id": "cohort_2024_q4_piano_adventures",
      "difference": "+5.2%",
      "significance": "p < 0.05"
    }
  ]
}
```

## Behavior and rules

### xAPI Statement Generation Rules
- **Real-time Generation**: xAPI statements generated immediately upon learning event completion
- **Compliance**: Full compliance with xAPI 1.0.3 specification and profiles
- **Statement Storage**: Statements stored in Learning Record Store (LRS) with 7-year retention
- **Privacy Protection**: Student identifiers anonymized in cross-organization analytics
- **Verb Consistency**: Use standard ADL verbs when available, custom verbs only for MLC-specific actions
- **Context Preservation**: Always include assignment, sequence, and session context in statements

### Predictive Model Rules
- **Model Versioning**: All predictive models versioned and tracked with performance metrics
- **Retraining Schedule**: Models retrained monthly with latest student outcome data
- **Confidence Thresholds**: Predictions below 0.7 confidence flagged as uncertain
- **Bias Detection**: Regular audits for demographic bias in predictive models
- **Human Oversight**: High-stakes predictions (e.g., at-risk identification) require human review
- **Feedback Loop**: Model predictions compared against actual outcomes for continuous improvement

### Learning Path Optimization Rules
- **Sample Size Threshold**: Correlation analysis requires minimum 100 students per path
- **Statistical Significance**: Only correlations with p < 0.05 used for recommendations
- **Personalization Balance**: Recommendations consider both data-driven insights and pedagogical principles
- **Teacher Override**: Teachers can always override algorithm recommendations
- **Adaptive Adjustment**: Learning paths automatically adjust based on student performance patterns
- **A/B Testing**: New learning path variations tested on control groups before broad deployment

### Privacy and Ethics Rules
- **Anonymization**: Cross-organization analytics use hashed identifiers
- **Consent Management**: Students/parents can opt out of advanced analytics (basic progress tracking remains)
- **Data Minimization**: Only collect and analyze data necessary for educational purposes
- **Transparency**: Students and teachers can view what analytics data is collected about them
- **Bias Mitigation**: Regular audits ensure analytics don't disadvantage any demographic groups
- **Research Ethics**: Any research use of analytics data requires IRB approval and informed consent

## UX requirements (if applicable)

### Analytics Dashboard Types

#### Teacher Analytics Dashboard
- **Student At-Risk Panel**: Visual indicators for students predicted to need intervention
- **Learning Path Insights**: Visualization of optimal learning paths based on cohort analysis
- **Concept Correlation View**: Interactive graph showing concept relationships and dependencies
- **Progress Prediction**: Timeline showing expected student completion dates
- **Intervention Recommendations**: Actionable suggestions based on predictive models

#### Admin Analytics Dashboard
- **Organization Health Score**: Composite metric combining engagement, retention, and outcomes
- **Cohort Comparison**: Side-by-side analysis of different student groups
- **Curriculum Effectiveness**: Analysis of sequence and assignment performance
- **Retention Prediction**: Early warning system for at-risk subscriptions
- **ROI Analysis**: Learning outcomes correlated with subscription investment

#### Research Analytics Portal
- **Anonymized Data Explorer**: Query interface for educational research
- **Statistical Analysis Tools**: Built-in tools for hypothesis testing and correlation analysis
- **Cohort Builder**: Create custom cohorts for comparative analysis
- **Visualization Library**: Pre-built charts and graphs for common research questions
- **Export Capabilities**: CSV/JSON export for external statistical analysis

### Key Interaction Patterns
- **Drill-down Navigation**: Click from high-level insights to detailed supporting data
- **Filter and Compare**: Flexible filtering to create custom analytical views
- **Time-range Selection**: Analyze trends across different time periods
- **Cohort Selection**: Compare different student populations
- **Export and Share**: Generate reports for stakeholders

### Real-time vs. Batch Processing
- **Real-time Metrics**: Student progress, engagement scores, at-risk flags (< 5 seconds latency)
- **Hourly Updates**: Learning velocity, session patterns, concept mastery trends
- **Daily Analysis**: Predictive models, correlation analysis, cohort comparisons
- **Weekly Reports**: Long-term trends, curriculum effectiveness, retention predictions
- **Monthly Analysis**: Model retraining, cross-population insights, research analytics

## Telemetry

### Analytics Events

#### Prediction Generated Events
```javascript
// Student success prediction generated
{
  event_type: "prediction_generated",
  properties: {
    model_id: "student_success_v2.1",
    student_id: "usr_12345",
    prediction_type: "success_probability",
    result: 0.87,
    confidence: 0.94,
    computation_time_ms: 145
  }
}

// Intervention recommendation triggered
{
  event_type: "intervention_recommended",
  properties: {
    student_id: "usr_12345",
    intervention_type: "review_reminder",
    trigger: "incomplete_reviews",
    priority: "medium",
    auto_action_taken: false
  }
}
```

#### Learning Path Optimization Events
```javascript
// Adaptive recommendation generated
{
  event_type: "adaptive_recommendation_generated",
  properties: {
    student_id: "usr_12345",
    recommendation_type: "game_suggestion",
    game_id: "G-02145",
    reason: "gap_fill",
    confidence: 0.85,
    accepted: true
  }
}

// Learning path adjusted
{
  event_type: "learning_path_adjusted",
  properties: {
    student_id: "usr_12345",
    sequence_id: "seq_456",
    adjustment_type: "difficulty_increase",
    reason: "consistent_overperformance",
    previous_level: 3,
    new_level: 4
  }
}
```

#### Research Analytics Events
```javascript
// Cohort analysis completed
{
  event_type: "cohort_analysis_completed",
  properties: {
    cohort_id: "cohort_2025_q1_piano_adventures",
    analysis_type: "mastery_rate_comparison",
    sample_size: 1247,
    computation_time_ms: 8450,
    significant_findings: 3
  }
}

// Model performance evaluation
{
  event_type: "model_performance_evaluated",
  properties: {
    model_id: "student_success_v2.1",
    accuracy: 0.87,
    precision: 0.84,
    recall: 0.89,
    f1_score: 0.86,
    sample_size: 5847
  }
}
```

## Permissions

### Analytics Access by Role

#### Teacher Role
- **Student Analytics**: Predictive insights and recommendations for their assigned students
- **Learning Path Insights**: Optimal learning paths and concept correlations
- **Intervention Alerts**: Notifications for at-risk students with suggested actions
- **Performance Predictions**: Estimated completion times and success probabilities
- **No Cross-Teacher Access**: Cannot see analytics for other teachers' students

#### Teacher-Admin Role
- **Extended Analytics**: All teacher analytics plus organization-wide insights
- **Cohort Analysis**: Compare different classes and teacher effectiveness
- **Curriculum Analytics**: Sequence and assignment performance across organization
- **Retention Predictions**: Early warning system for at-risk subscriptions
- **Teacher Performance**: Anonymized comparison of teaching effectiveness

#### Subscriber-Admin Role
- **Organization Analytics**: Complete analytics for their organization
- **ROI Analysis**: Learning outcomes correlated with subscription investment
- **Retention Predictions**: Subscription churn risk and engagement metrics
- **Curriculum Effectiveness**: Which sequences and games work best for their students
- **Export Capabilities**: Full data export for their organization

#### System-Admin Role
- **Global Analytics**: Cross-organization insights (anonymized)
- **Platform Research**: Educational effectiveness research across all users
- **Model Management**: Access to predictive model performance and tuning
- **A/B Testing**: Design and analyze platform experiments
- **Full Research Access**: Complete anonymized data for platform improvement

#### Research Partner Role (future)
- **Anonymized Data Access**: IRB-approved research queries
- **Statistical Tools**: Built-in analysis and visualization tools
- **Cohort Builder**: Create custom cohorts for research questions
- **Export Capabilities**: Data export for external analysis
- **No PII Access**: Strictly anonymized data only

### Data Privacy and Ethical Safeguards
- **Informed Consent**: Students/parents informed about analytics data collection
- **Opt-out Rights**: Can opt out of advanced analytics while maintaining basic progress tracking
- **Transparency**: Students can view what predictions and insights are generated about them
- **Bias Audits**: Regular audits to ensure analytics don't disadvantage any groups
- **Human Oversight**: High-stakes decisions require human review, not just algorithms
- **Data Retention**: Analytics data retained for 7 years, then anonymized or deleted

## Supporting Documents Referenced

This learning analytics framework specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game scoring, mastery criteria, and learning element structure for analytics tracking
- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — Learning philosophy and quantitative assessment approach that guides analytics design
- [Assignment Play Re-design.docx.txt](../00-ORIG-CONTEXT/Assignment%20Play%20Re-design.docx.txt) — Score tracking and progress display requirements for analytics dashboards
- [MLC SeqCreate 2020.xlsx - LIFE SEQ.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20LIFE%20SEQ.csv) — Lifetime Musician sequence structure for learning path analysis
- [Level1CurriculumMap.xls - Sheet1.csv](../00-ORIG-CONTEXT/Level1CurriculumMap.xls%20-%20Sheet1.csv) — Curriculum progression for concept correlation analysis

## Dependencies

### Core System Dependencies
- **Event Model**: Raw event data foundation from [event-model.md](./event-model.md)
- **Game Model**: Game metadata and learning concepts from [game-model.md](../08-games-registry-and-authoring/game-model.md)
- **Sequence Model**: Learning path structure from [sequence-model.md](../09-sequences-and-curriculum-tools/sequence-model.md)
- **Assignment Model**: Assignment structure and progression from [assignment-model.md](../07-content-architecture/assignment-model.md)
- **Roles and Permissions**: Access control from [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md)

### Analytics Infrastructure Dependencies
- **Learning Record Store (LRS)**: xAPI statement storage and retrieval
- **Machine Learning Pipeline**: Model training, evaluation, and deployment infrastructure
- **Time-Series Database**: Historical event data for trend analysis
- **Data Warehouse**: Aggregated data for complex analytics queries
- **Real-time Processing**: Stream processing for live predictions and alerts
- **Statistical Computing**: R/Python environments for advanced analysis

### Data Source Dependencies
- **Student Progress Data**: Assignment completion, game scores, mastery achievements
- **Engagement Metrics**: Session frequency, duration, consistency patterns
- **Content Performance**: Game effectiveness, sequence completion rates
- **User Behavior**: Navigation patterns, feature usage, interaction logs
- **External Benchmarks**: Educational research data for comparative analysis

### Related Documentation
- [Event Model](./event-model.md) - Event tracking and data collection
- [Teacher Reports KPIs](./teacher-reports-kpis.md) - Standard teacher-facing metrics
- [Admin Reports KPIs](./admin-reports-kpis.md) - Administrative analytics and reporting
- [Sysadmin Ops Dashboards](./sysadmin-ops-dashboards.md) - Operational monitoring
- [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md) - Progress tracking logic
- [Retention Review Logic](../10-assignments-engine/retention-review-logic.md) - Spaced repetition algorithms

## Open questions

### xAPI Implementation
- **LRS Provider**: Should we host our own LRS or use a third-party service (e.g., Learning Locker, Watershed)?
- **Statement Volume**: What's the expected xAPI statement volume at scale and what are the storage implications?
- **External Sharing**: Should organizations be able to export xAPI statements to their own LRS?
- **Profile Extensions**: What additional xAPI profiles should we implement (cmi5, Video, etc.)?

### Predictive Analytics
- **Model Selection**: Which machine learning algorithms work best for student success prediction (random forests, neural networks, ensemble methods)?
- **Feature Engineering**: What additional features would improve prediction accuracy?
- **Intervention Effectiveness**: How do we measure whether our recommended interventions actually work?
- **False Positive Management**: How do we minimize false at-risk predictions that could stigmatize students?

### Learning Path Optimization
- **Personalization Limits**: How much can we personalize without fragmenting the curriculum?
- **Teacher Authority**: When should algorithms override teacher-assigned sequences?
- **Control Groups**: How do we ethically conduct A/B testing on educational interventions?
- **Optimal Path Discovery**: How do we identify truly optimal learning paths vs. simply popular ones?

### Privacy and Ethics
- **Algorithmic Transparency**: How much should we reveal about how predictions are made?
- **Bias Detection**: What metrics indicate demographic bias in our models?
- **Consent Granularity**: Should students be able to opt out of specific analytics types?
- **Research Ethics**: What additional IRB considerations apply to educational research?

### Technical Implementation
- **Real-time Requirements**: What's the acceptable latency for predictive analytics in live dashboards?
- **Model Retraining**: How often should models be retrained and how do we handle model versioning?
- **Computational Costs**: What's the infrastructure cost for running advanced analytics at scale?
- **Data Quality**: How do we ensure analytics data quality and handle missing or inconsistent data?

### Business Intelligence
- **ROI Measurement**: How do we quantify the value of advanced analytics to subscribers?
- **Feature Tiering**: Should advanced analytics be a premium feature or available to all tiers?
- **Research Partnerships**: Should we partner with educational institutions for research validation?
- **Competitive Advantage**: What analytics capabilities differentiate MLC from competitors?
