 public async Task<List<BenchmarkGroupedData>> GetAffiliateBenchmark(GetAffiliateBenchmarkRequest request)
        {
            List<BenchmarkGroupedData> groupData = new List<BenchmarkGroupedData>();
            //convert string to datetime
            var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
            var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

            //convert format as yyyymm 202508
            var formatedStartdate = startDateTime.ToString("yyyyMM");
            var formatedEnddate = endDateTime.ToString("yyyyMM");


            //convert format as yyyy-mm-dd 2025-08-01
           var  formatedStartdateyymmdd = startDateTime.ToString("yyyy-MM-dd");
           var formatedEnddateyymmdd = endDateTime.ToString("yyyy-MM-dd");

            AffiliateBenchmarkQuery affiliatebenchmarkquery = new AffiliateBenchmarkQuery(); 
            List<AffiliateBenchmark> GlobalResponseList = affiliatebenchmarkquery.affiliateDataList.ToList();
            var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
            var kpiinfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);
            var sqlAffiliateName = await _benchMarkRepository.GetAffiliateLists();
            var affiliateds = request.affiliateId?.Split(',').Select(int.Parse).ToList();

            var affiliateList = new List<GetCaseHierarchyResponse>();
            foreach (var item in affiliateds!)
            {
                var lstPlants = await _benchMarkRepository.GetCaseHierarchyAsync(item);
                affiliateList.AddRange(lstPlants);
            }


            foreach (var item in GlobalResponseList.Where(a => a.kpiCode == request.kpiCode))
            {
                var actual = AffilateReplaceQuery(item.query!, formatedStartdateyymmdd, formatedEnddateyymmdd, formatedStartdate, formatedEnddate, "", request);
                var queryBestAchievedEver = AffilateReplaceQuery(item.bestAchievedEver!, formatedStartdateyymmdd, formatedEnddateyymmdd, formatedStartdate, formatedEnddate, "", request);


                var actualQuery = await _benchMarkRepository.ExecuteBigDataQuery<BenchmarkGroupedData>(actual, reader =>
                {
                    int affilateIdFromBigData = reader["affiliate_id"] != DBNull.Value ? Convert.ToInt32(reader["affiliate_id"]) : 0;
                    string? affilateName = sqlAffiliateName.Find(x => x.affiliateId == affilateIdFromBigData)?.affiliateName;
                    return new BenchmarkGroupedData
                    {
                       
                        affiliateName = affilateName,
                        actual = reader["actual"] != DBNull.Value ? Convert.ToDecimal(reader["actual"]) : 0,
                        absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,                        
                    };
                }) ?? new List<BenchmarkGroupedData>();
                var bestAchievedResult = await _benchMarkRepository.ExecuteBigDataQuery<BenchmarkGroupedData>(queryBestAchievedEver, reader =>
                {
                    int affilateIdFromBigData = reader["affiliate_id"] != DBNull.Value ? Convert.ToInt32(reader["affiliate_id"]) : 0;
                    string? affilateName = sqlAffiliateName.Find(x => x.affiliateId == affilateIdFromBigData)?.affiliateName;
                    return new BenchmarkGroupedData
                    {
                        bestAchievedEver = reader["best_achieved"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved"]) : 0,
                        affiliateName = affilateName
                    };
                }) ?? new List<BenchmarkGroupedData>();

                 groupData = (from a in actualQuery
                              join b in bestAchievedResult
                                 on a.affiliateName equals b.affiliateName into temp
                                 from b in temp.DefaultIfEmpty()
                                 select new BenchmarkGroupedData
                                 {
                                     affiliateName= a.affiliateName,
                                     actual= a.actual,
                                     absolute= a.absolute,
                                     bestAchievedEver=b?.bestAchievedEver ?? 0
                                 }).GroupBy(x=>x.affiliateName).Select(g=> new BenchmarkGroupedData
                                 { 
                                     affiliateName=g.Key,
                                     actual=g.Sum(x=>x.actual),
                                     absolute= g.Sum(x=>x.absolute),
                                     bestAchievedEver=g.Max(x=>x.bestAchievedEver)
                                 }
                                 ).ToList();


                decimal bestAchivedEverMin;
                decimal bestAchivedForSinglePeriod;

                    if (kpiinfo?.direction == 1)
                    {
                    bestAchivedEverMin = (groupData != null && groupData.Count > 0) ? groupData.Min(x => x.bestAchievedEverMin) : 0;
                    bestAchivedForSinglePeriod = (groupData != null && groupData.Count > 0) ? groupData.Min(x => x.actual) : 0;
                    
                    }
                    else
                    {
                        bestAchivedEverMin = groupData.Max(x => x.bestAchievedEver);
                        bestAchivedForSinglePeriod = groupData.Max(x => x.actual);
                    }


                if (actualQuery == null || !actualQuery.Any())
                {
                    return new List<BenchmarkGroupedData>();
                }
                if (queryBestAchievedEver == null || queryBestAchievedEver.Length == 0)
                {
                    return new List<BenchmarkGroupedData>();
                }




                     int? sqlDirection = sqlKpiDataList.Where(x => x.kpiCode == request.kpiCode).Select(x => x.direction).FirstOrDefault();
                    long? targetMin = sqlKpiDataList.Where(x => x.kpiCode == request.kpiCode).Select(x => x.overallTargetMin).FirstOrDefault();
                    long? targetMax = sqlKpiDataList.Where(x => x.kpiCode == request.kpiCode).Select(x => x.overallTargetMax).FirstOrDefault();
                    foreach (var itemData in groupData!)
                    {

                        if (itemData.actual >= targetMin && itemData.actual <= targetMax)
                        {
                            itemData.state = 1;
                        }
                        else
                        {
                            itemData.state = 0;
                        }


                        itemData.direction = sqlDirection;
                        itemData.target = targetMax!;
                        itemData.bestAchievedEverMin = bestAchivedEverMin;
                        itemData.bestAchievedForSinglePeriod = bestAchivedForSinglePeriod;

                    }

                

            }

            Method has 10 parameters, which is greater than the 7 authorized.
            Refactor this method to reduce its Cognitive Complexity from 18 to the 15 allowed.






            return groupData!;
        }
