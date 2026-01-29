# On-Call Responsibilities

This document outlines the responsibilities and procedures for engineers on on-call duty for the Swipe iOS application.

## Overview

The on-call engineer is the first line of response for critical issues, customer tickets, and production incidents. This role rotates among team members to ensure continuous coverage and shared knowledge.

## Intercom Ticket Management

We use Intercom for on-call requests. When a ticket comes in:

1. **Acknowledge the ticket immediately** - Let the customer know you've received their request
2. **Assess the severity** - Determine if it's a critical issue, bug, or feature request
3. **Start working on the ticket** - Begin investigating and resolving the issue
4. **Communicate progress** - Keep the customer updated on your progress
5. **Release a build with the fix** - Once the issue is fixed, follow the release process documented in [Release Management](release-management.md)

### Ticket Priority Levels

- **🟢 Customer Tickets**: Highest priority - these should be addressed first
- **🔴 Critical Crashes**: Affecting multiple users or core functionality
- **🟡 Non-critical Issues**: Can be scheduled for next release cycle

## On-Call Handover Sheet

The on-call engineer is responsible for filling out the **On-Call Handover Sheet** on Slack (Stany Bot) every **Monday at 10:30 AM**.

### Handover Metrics

Please include the following metrics in your handover:

```
Total Crash Counts
→ Number of crashes recorded from Monday (last week) to Sunday. (Crashlytics -> Dashboard -> Date Filters)

Crash Events
→ Total number of Firebase events related to crashes (count all relevant events).

Resolved Events
→ Number of events that have been fixed or are scheduled for release in upcoming builds.

Total Customer Tickets (Intercom)
→ Number of customer tickets received on Intercom.
🟢 These should be treated as highest priority.

Pending Customer Tickets for Next On-Call
→ List any tickets that remain unresolved due to dependencies or pending fixes.
→ The next on-call engineer should be aware of their status and verify for possible regressions.

Blockers / Notes for Next On-Call
→ Mention any blockers, context, or useful information that would help the next on-call engineer continue smoothly.
```

## Metrics Sheet

Fill this sheet on every **Thursday and Monday** of the week:

**Metrics Sheet URL**: https://docs.google.com/spreadsheets/d/1ctuXOtHjlkggUb4gqocYCzGF4TxLyJEFEZdBiCYuwVY/edit?gid=272024554#gid=272024554

### What to Track

- Crash counts and trends
- Customer ticket volume
- Resolution times
- Build releases
- Critical issues encountered
- Performance metrics

## Daily Responsibilities

### Morning (Start of Day)
- [ ] Check Intercom for new tickets
- [ ] Review Firebase Crashlytics dashboard
- [ ] Check App Store Connect for reviews and issues
- [ ] Review any alerts or notifications

### Throughout the Day
- [ ] Monitor Intercom for new tickets
- [ ] Respond to customer inquiries within 2 hours
- [ ] Track crashes and investigate patterns
- [ ] Communicate with team about critical issues
- [ ] Update ticket status and handover notes

### End of Day
- [ ] Update handover notes with progress
- [ ] Ensure all critical tickets are acknowledged
- [ ] Document any blockers for next on-call
- [ ] Prepare fixes for next build if needed

## Weekly Responsibilities

### Monday at 10:30 AM
- [ ] Fill out On-Call Handover Sheet on Slack (Stany Bot)
- [ ] Fill out Metrics Sheet
- [ ] Review handover from previous on-call engineer
- [ ] Address any pending tickets from last week

### Thursday
- [ ] Fill out Metrics Sheet
- [ ] Review crash trends for the week
- [ ] Prepare summary of issues for team standup

## Escalation Process

### When to Escalate

Escalate to the team lead or engineering manager when:
- Critical crash affecting > 5% of users
- Security vulnerability discovered
- Data loss or corruption issues
- Payment processing failures
- Legal or compliance concerns
- Customer escalation to management

### How to Escalate

1. Post in `#swipe-ios` Slack channel
2. Tag relevant team members
3. Provide clear description of the issue
4. Include impact assessment and urgency level
5. Suggest immediate next steps if known

## Hotfix Process

If you identify a critical issue requiring a hotfix:

1. **Create hotfix branch** from staging
2. **Fix the issue** with minimal, focused changes
3. **Test thoroughly** - verify the fix works
4. **Follow expedited release process** - refer to [Release Management](release-management.md#hotfix-process)
5. **Request expedited review** in App Store Connect
6. **Monitor closely** after release

## Tools and Access

Ensure you have access to:
- **Intercom**: Customer support tickets
- **Firebase Crashlytics**: Crash reporting
- **App Store Connect**: App reviews and analytics
- **Slack**: Team communication (Stany Bot)
- **Metrics Sheet**: Google Sheets tracking
- **Xcode Organizer**: Crash logs and diagnostics

## Best Practices

1. **Respond quickly** - Acknowledge tickets within 1 hour during business hours
2. **Communicate clearly** - Keep customers and team informed
3. **Document everything** - Update handover notes and tickets
4. **Prioritize wisely** - Critical issues first, then customer tickets
5. **Ask for help** - Don't hesitate to involve the team
6. **Stay proactive** - Monitor dashboards regularly
7. **Be thorough** - Verify fixes before releasing

## Handover Checklist

When transitioning to the next on-call engineer:

- [ ] Fill out On-Call Handover Sheet on Slack
- [ ] Update Metrics Sheet
- [ ] Document all pending tickets
- [ ] Note any known issues or patterns
- [ ] Share context on any blockers
- [ ] Ensure all critical tickets are addressed or handed over
- [ ] Brief next on-call engineer on any urgent matters

## Contact Information

For urgent issues outside of your expertise:
- **Engineering Lead**: [Contact via Slack]
- **Product Manager**: [Contact via Slack]
- **Backend Team**: [Contact via #backend channel]

## Related Documentation

- [Release Management](release-management.md) - For releasing fixes and hotfixes
- [Git Workflow](git-workflow.md) - For branch management and PRs
- [Troubleshooting](troubleshooting.md) - For common issues and solutions

---

[← Previous: Release Management](release-management.md) | [Back to Index](../README.md) | [Next: Tools & Resources →](tools-resources.md)
