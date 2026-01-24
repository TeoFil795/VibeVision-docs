# WodVision Documentation

Central documentation repository for the WodVision CrossFit movement analysis platform.

## 📚 Documentation Index

### Quick Reference
- **[claude.md](claude.md)** - Main development guide (v1.0.12)
  - System prompt for AI assistants
  - Quick reference for all components
  - API endpoints, database schema summaries
  - Security checklist, git workflow
  - TODO list and roadmap

### Comprehensive Technical Docs
- **[DOCUMENTAZIONE_TECNICA_WODVISION.md](DOCUMENTAZIONE_TECNICA_WODVISION.md)** - Frontend + Laravel + Database
  - Complete Flutter app architecture
  - Laravel backend API reference
  - Database schema (all 15+ tables)
  - Authentication flow diagrams

- **[DOCUMENTAZIONE_BACKEND_PYTHON_WODVISION.md](DOCUMENTAZIONE_BACKEND_PYTHON_WODVISION.md)** - AI Processing
  - MediaPipe pose detection deep dive
  - YOLO object detection models
  - Gemini AI integration guide
  - Movement criteria knowledge base (35+ exercises)

### Development Principles
- **[CODING_PRINCIPLES.md](CODING_PRINCIPLES.md)** - Code quality standards
  - Pragmatic quality approach
  - DRY, KISS, YAGNI principles
  - Code review checklist

- **[SECURITY_PRINCIPLES.md](SECURITY_PRINCIPLES.md)** - Security best practices
  - OWASP Top 10 coverage
  - Credential management
  - Input validation patterns
  - API security guidelines

- **[DESIGN_PRINCIPLES.md](DESIGN_PRINCIPLES.md)** - UI/UX guidelines
  - Inspired by Stripe, Airbnb, Linear
  - Component patterns
  - Accessibility standards

### Session Logs (Build in Public)
- **[SESSION_LOG_2026-01-21_ANALYTICS.md](SESSION_LOG_2026-01-21_ANALYTICS.md)** - Firebase Analytics implementation
  - 52+ events tracked (92% coverage)
  - Funnel setup (Signup, Video Journey, Revenue, Referral)
  - User properties & segmentation

- **[SESSION_LOG_2026-01-17.md](SESSION_LOG_2026-01-17.md)** - RevenueCat integration
  - Paywall UI implementation
  - Subscription status fixes
  - Post-purchase logout bug resolution

- **[SESSION_LOG_2026-01-15.md](SESSION_LOG_2026-01-15.md)** - Security audit
  - Token encryption (flutter_secure_storage)
  - HTTPS enforcement
  - Secret Manager migration

### Migration Plans
- **[MIGRATION_PLAN_REVENUECAT.md](MIGRATION_PLAN_REVENUECAT.md)** - Legacy subscriptions migration
  - Database audit results (135 users, 3 active subscriptions)
  - RevenueCat import strategy
  - Webhook sync setup
  - Zero-downtime deployment plan

## 🏗️ Project Structure

```
WodVision Platform/
├── wodvision-mobile/     # Flutter app (Dart 3.5.3+, Provider)
├── wodvision-api/        # Laravel 11 REST API (MySQL, Sanctum)
├── wodvision-ai/         # Python AI backend (FastAPI, MediaPipe, Gemini)
└── wodvision-docs/       # This repository
```

## 🚀 Quick Start

1. **Setup Development Environment**:
   - Read `claude.md` for overview
   - Check `SECURITY_PRINCIPLES.md` for credential management
   - Review `CODING_PRINCIPLES.md` for standards

2. **Understand Architecture**:
   - Mobile app architecture: `DOCUMENTAZIONE_TECNICA_WODVISION.md` Section 1-2
   - API backend: `DOCUMENTAZIONE_TECNICA_WODVISION.md` Section 3-4
   - AI backend: `DOCUMENTAZIONE_BACKEND_PYTHON_WODVISION.md`

3. **Start Developing**:
   - Follow git workflow in `claude.md` Section 10
   - Refer to API endpoints in `claude.md` Section 5
   - Check TODO list in `claude.md` Section 9

## 📊 Current Status (v1.0.12)

**Completed**:
- ✅ RevenueCat subscription management
- ✅ Firebase Analytics (92% coverage)
- ✅ Security audit (HTTPS, token encryption, Secret Manager)
- ✅ 135 users, 3 active subscriptions
- ✅ 35+ CrossFit movements supported

**In Progress**:
- 🔄 Legacy subscription migration (1 real customer)
- 🔄 Storage cleanup automation
- 🔄 AI validation layer

**Planned**:
- 📅 Google Play Store release
- 📅 BigQuery analytics export
- 📅 AI prompt optimization

## 🔗 External Resources

- **Firebase Console**: [console.firebase.google.com](https://console.firebase.google.com/project/wodvision-52d46)
- **GCP Console**: [console.cloud.google.com](https://console.cloud.google.com) (project: peak-ascent-452414-k2)
- **RevenueCat Dashboard**: [app.revenuecat.com](https://app.revenuecat.com)
- **DigitalOcean**: cloud.digitalocean.com (server: 64.226.127.138)

## 🤝 Contributing

This is a private project. For development guidelines:
1. Always read `claude.md` before making changes
2. Follow security checklist before commits
3. Update relevant docs when modifying architecture
4. Add session logs for major feature implementations

## 📞 Support

For questions or issues, refer to the appropriate documentation:
- **Frontend issues**: `DOCUMENTAZIONE_TECNICA_WODVISION.md` Section 1-2
- **API issues**: `DOCUMENTAZIONE_TECNICA_WODVISION.md` Section 3-4
- **AI issues**: `DOCUMENTAZIONE_BACKEND_PYTHON_WODVISION.md`
- **General questions**: `claude.md`

---

*Last updated: 2026-01-24 (v1.0.12)*
