def maximumPeople(p, x, y, r):
    # number of towns and clouds
    n = len(x)
    m = len(y)

    # Create events list.
    # Each event is a tuple: (position, event_type, index)
    # event_type: 0 for cloud start, 1 for town, 2 for cloud end.
    # Ordering by event_type in this way ensures that if events share the same position:
    #  - cloud start events are processed first (cloud becomes active before reaching a town)
    #  - then town events (town is processed with the current active clouds)
    #  - then cloud end events.
    events = []

    # Add cloud start and end events.
    for i in range(m):
        # The cloud covers from (y[i] - r[i]) to (y[i] + r[i])
        L = y[i] - r[i]
        R = y[i] + r[i]
        events.append((L, 0, i))  # cloud i starts at L
        events.append((R, 2, i))  # cloud i ends at R

    # Add town events. We'll store town index to later retrieve its population.
    for i in range(n):
        events.append((x[i], 1, i))  # town event at x[i]

    # Sort events.
    # Python sorts tuples lexicographically, so this sorts primarily by position,
    # and in case of a tie, by event_type (0, then 1, then 2).
    events.sort()

    baseline = 0  # People who are always sunny (towns not covered by any cloud)
    uniqueContrib = [0] * m  # uniqueContrib[i] holds the rescued population if cloud i is removed
    active = set()  # active clouds covering the current position

    # Process all events in sorted order.
    for pos, typ, idx in events:
        if typ == 0:
            # Cloud start event.
            active.add(idx)
        elif typ == 1:
            # Town event.
            # idx is town index.
            if len(active) == 0:
                baseline += p[idx]
            elif len(active) == 1:
                # The town is covered by exactly one cloud.
                # Find that cloud (the only element in 'active').
                unique_cloud = next(iter(active))
                uniqueContrib[unique_cloud] += p[idx]
        else:  # typ == 2, cloud end event.
            # Cloud idx ends coverage.
            # Use discard in case of any discrepancy.
            active.discard(idx)

    # The best improvement is achieved by removing the cloud with the highest unique contribution.
    rescue = max(uniqueContrib) if m > 0 else 0

    return baseline + rescue

if __name__ == '__main__':
    import os
    fptr = open(os.environ['OUTPUT_PATH'], 'w')

    n = int(input().strip())
    p = list(map(int, input().rstrip().split()))
    x = list(map(int, input().rstrip().split()))

    m = int(input().strip())
    y = list(map(int, input().rstrip().split()))
    r = list(map(int, input().rstrip().split()))

    result = maximumPeople(p, x, y, r)

    fptr.write(str(result) + '\n')
    fptr.close()
# PROJECT-_CLOUD-_COVER
