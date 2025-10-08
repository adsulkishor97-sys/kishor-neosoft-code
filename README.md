 bestAchivedEverMin = (groupData != null && groupData.Any()) ? groupData.Min(x => x.bestAchievedEverMin) : 0;


 Prefer comparing 'Count' to 0 rather than using 'Any()', both for clarity and for performance
