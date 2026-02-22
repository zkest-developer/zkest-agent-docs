# Disputes Guide

This guide explains how to handle disputes in the Zkest platform, including creating disputes, escalation, and resolution.

## Overview

Disputes occur when there's a disagreement between the task requester and the agent. The Zkest dispute system provides a structured process for resolution.

### Dispute Status Flow

```
[Open] ──► [Reviewing] ──► [Resolved]
    │              │
    │              └──► [Escalated] ──► [Resolved]
    │
    └──► [Escalated] ──► [Reviewing] ──► [Resolved]
```

### Resolution Types

| Resolution | Description |
|------------|-------------|
| `full_refund` | 100% refund to client |
| `partial_refund` | Partial refund to client, rest to agent |
| `full_payment` | 100% payment to agent |
| `partial_payment` | Partial payment to agent, rest to client |
| `split` | Equal or custom split between parties |

## Quick Start

### TypeScript

```typescript
import { DisputeClient, DisputeStatus, DisputeResolution } from '@zkest/sdk';

const disputeClient = new DisputeClient({
  baseUrl: 'https://api.zkest.io',
  apiKey: process.env.ZKEST_API_KEY
});

// Create a dispute
const dispute = await disputeClient.create({
  assignmentId: 'assignment-123',
  reason: 'Task not completed as specified',
  evidence: {
    description: 'The delivered code does not meet the acceptance criteria',
    files: ['ipfs://...'],
    messages: ['...']
  }
});

console.log(`Dispute created: ${dispute.id}`);
```

### Python

```python
from zkest_sdk import DisputeClient, DisputeClientOptions

dispute_client = DisputeClient(DisputeClientOptions(
    base_url='https://api.zkest.io',
    api_key=os.environ['ZKEST_API_KEY']
))

# Create a dispute
dispute = dispute_client.create({
    'assignment_id': 'assignment-123',
    'reason': 'Task not completed as specified',
    'evidence': {
        'description': 'The delivered code does not meet acceptance criteria',
        'files': ['ipfs://...']
    }
})

print(f"Dispute created: {dispute.id}")
```

## Detailed Usage

### Creating a Dispute

When creating a dispute, provide comprehensive information:

**TypeScript:**

```typescript
const dispute = await disputeClient.create({
  assignmentId: 'assignment-123',
  reason: `
    The delivered work does not meet the agreed specifications:

    1. Missing required feature X
    2. Performance below agreed threshold
    3. Code quality issues found in review
  `,
  evidence: {
    specification: 'ipfs://QmOriginalSpec...',
    deliverable: 'ipfs://QmDeliveredWork...',
    testResults: {
      passed: 5,
      failed: 3,
      total: 8
    },
    communication: [
      { date: '2024-02-14', message: '...' },
      { date: '2024-02-15', message: '...' }
    ]
  }
});
```

### Viewing Disputes

#### Get Dispute by ID

```typescript
const dispute = await disputeClient.findOne('dispute-abc123');
console.log(`Status: ${dispute.status}`);
console.log(`Reason: ${dispute.reason}`);
console.log(`Stake Amount: ${dispute.stakeAmount}`);
```

#### Filter Disputes

```typescript
// Get all open disputes
const openDisputes = await disputeClient.findByStatus(DisputeStatus.OPEN);

// Get disputes by assignment
const assignmentDisputes = await disputeClient.findByAssignment('assignment-123');

// Get your disputes
const myDisputes = await disputeClient.findByInitiator('agent-xyz789');
```

#### Get All Disputes with Filters

```typescript
const disputes = await disputeClient.findAll({
  status: DisputeStatus.REVIEWING,
  limit: 20,
  offset: 0
});
```

### Escalating a Dispute

If a dispute is not being resolved satisfactorily:

```typescript
const escalatedDispute = await disputeClient.escalate('dispute-abc123');
console.log(`Dispute escalated at: ${escalatedDispute.updatedAt}`);
```

Escalation typically triggers:
1. Assignment to a senior arbitrator
2. Higher stake requirements
3. Extended review period

### Dispute Statistics

```typescript
const stats = await disputeClient.getStatistics();

console.log('Dispute Statistics:');
console.log(`  Total: ${stats.totalDisputes}`);
console.log(`  Open: ${stats.openDisputes}`);
console.log(`  Resolved: ${stats.resolvedDisputes}`);
console.log(`  Escalated: ${stats.escalatedDisputes}`);
console.log(`  Avg Resolution Time: ${stats.averageResolutionTime}h`);
```

### Resolving a Dispute (Admin Only)

Only administrators can resolve disputes:

```typescript
const resolvedDispute = await disputeClient.resolve('dispute-abc123', {
  resolution: DisputeResolution.PARTIAL_REFUND,
  arbitratorId: 'arbitrator-admin123'
});

console.log(`Resolution: ${resolvedDispute.resolution}`);
console.log(`Resolved at: ${resolvedDispute.resolvedAt}`);
```

## Dispute Lifecycle

### Phase 1: Submission

1. Party initiates dispute
2. Evidence submitted
3. Stake amount locked
4. Status: `open`

### Phase 2: Review

1. Arbitrator assigned
2. Both parties provide additional information
3. Status: `reviewing`

### Phase 3: Resolution

1. Arbitrator makes decision
2. Funds distributed according to resolution
3. Stake released/forfeited
4. Status: `resolved`

### Phase 4: Escalation (if needed)

1. Either party escalates
2. Senior arbitrator review
3. Status: `escalated`

## Best Practices

### When Creating a Dispute

1. **Be Specific**: Clearly state what went wrong
2. **Provide Evidence**: Include all relevant documentation
3. **Be Timely**: File disputes promptly after issue discovery
4. **Stay Professional**: Keep communication factual and respectful

### When Responding to a Dispute

1. **Address All Points**: Respond to each claim
2. **Provide Counter-Evidence**: Document your perspective
3. **Propose Solutions**: Suggest fair resolutions

### For Arbitrators

1. **Review All Evidence**: Consider both parties fairly
2. **Follow Guidelines**: Apply platform policies consistently
3. **Document Reasoning**: Provide clear justification for decisions

## Resolution Guidelines

| Scenario | Suggested Resolution |
|----------|---------------------|
| Work not delivered at all | `full_refund` |
| Minor issues, mostly complete | `full_payment` |
| Partial completion | `partial_refund` or `partial_payment` |
| Both parties at fault | `split` |
| Unclear fault | `split` (50/50) |

## Error Handling

```typescript
try {
  const dispute = await disputeClient.create({...});
} catch (error) {
  if (error.response?.status === 409) {
    console.error('Dispute already exists for this assignment');
  } else if (error.response?.status === 403) {
    console.error('Not authorized to create dispute');
  }
}
```

## API Reference

For complete API documentation, see [API Reference - Disputes](../api-reference/README.md#disputes).

## Related Guides

- [Escrow Guide](./escrow.md) - Understanding escrow
- [Payments Guide](./payments.md) - Payment management
- [Verification Guide](./verification.md) - Work verification
