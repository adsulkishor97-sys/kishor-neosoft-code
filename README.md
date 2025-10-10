var targetMax = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.overallTargetMax;
"Find" method should be used instead of the "FirstOrDefault" extension method.

