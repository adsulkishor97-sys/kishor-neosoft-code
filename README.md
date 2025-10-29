 public async Task<IDictionary<AFSummaryTypes, AFValue>> SummaryAsync(AFTimeRange timeRange, AFSummaryTypes summaryType, AFCalculationBasis calculationBasis, AFTimestampCalculation timeType, CancellationToken cancellationToken = default(CancellationToken))
 {
     if (summaryType == AFSummaryTypes.None)
     {
         throw new ArgumentOutOfRangeException("summaryType");
     }

     if ((object)timeRange == null)
     {
         throw new ArgumentNullException("timeRange");
     }

     AFTimeSpan aFTimeSpan = new AFTimeSpan(timeRange.Span);
     bool reverseTime = timeRange.StartTime > timeRange.EndTime;
     IList<AFTimeIntervalDefinition> evenTimeIntervalDefinitions = aFTimeSpan.GetEvenTimeIntervalDefinitions(timeRange);
     IDictionary<AFSummaryTypes, AFValues> obj = await SummariesAsync(evenTimeIntervalDefinitions, reverseTime, summaryType, calculationBasis, timeType, cancellationToken).ConfigureAwait(continueOnCapturedContext: false);
     cancellationToken.ThrowIfCancellationRequested();
     IDictionary<AFSummaryTypes, AFValue> dictionary = new Dictionary<AFSummaryTypes, AFValue>();
     foreach (KeyValuePair<AFSummaryTypes, AFValues> item in obj)
     {
         if (item.Value.Count > 0)
         {
             dictionary.Add(item.Key, item.Value[0]);
         }
     }

     return dictionary;
 }
