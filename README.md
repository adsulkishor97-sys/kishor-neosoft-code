public async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmark(GetAssetBenchmarkRequest request)
        {
            //convert string to datetime
            var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
            var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

            //convert format as yyyymm 202508
            var formatedStartdate = startDateTime.ToString("yyyyMM");
            var formatedEnddate = endDateTime.ToString("yyyyMM");

            //convert format as yyyy-mm-dd 2025-08-01
            var formatedStartdateyymmdd = startDateTime.ToString("yyyy-MM-dd");
            var formatedEnddateyymmdd = endDateTime.ToString("yyyy-MM-dd");
            AssetBenchmarkQuery assetbenchmarkquery = new AssetBenchmarkQuery();
            List<AffiliateBenchmark> FileGlobalResponseList = assetbenchmarkquery.plantDataList.ToList();
            var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
            var kpiinfo = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == request.kpiCode);
            string quatedpmcode = "";
            // get plantIds based on affilateId
            var sapIds = request.sapId?.Split(',').Select(int.Parse).ToList();
            var formatSapIds = string.Join(",", sapIds!.Select(x => x.ToString().Length<18 ? $"'000000000{x}'" : $"'{x}'") );

           

            var tasks = FileGlobalResponseList
                .AsParallel()
                .Where(a => a.kpiCode == request.kpiCode)
                .Select(async item =>
                {
                    var queryBestAchievedEver = ReplaceAssetQuery(item.bestAchievedEver!, formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd, "", request, quatedpmcode, formatSapIds);
                    var bestAchievedresult = await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(queryBestAchievedEver, reader => new AssetBenchmarkGroupedData
                    {
                        manufacturer = reader["manufacturer"] != DBNull.Value ? Convert.ToString(reader["manufacturer"]) : "",
                        modelNumber = reader["model_number"] != DBNull.Value ? Convert.ToString(reader["model_number"]) : "",
                        bestAchievedEver = reader["best_achieved_ever"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved_ever"]) : 0,
                        absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
                        actual = reader["actual"] != DBNull.Value ? Convert.ToInt32(reader["actual"]) : 0,
                        sapId = reader["sap_id"] != DBNull.Value ? Convert.ToString(reader["sap_id"])!.TrimStart('0') : "",

                    }) ?? new List<AssetBenchmarkGroupedData>();
                    return bestAchievedresult;


                });
            var bestAchievedresults = await Task.WhenAll(tasks);
            var groupresultdata = bestAchievedresults.SelectMany(x => x).ToList();

            decimal bestAchivedEverMin;
            decimal bestAchivedForSinglePeriod;

                if (kpiinfo?.direction == 1)
                {
                    bestAchivedEverMin = groupresultdata.Min(x => x.bestAchievedEver);
                    bestAchivedForSinglePeriod = groupresultdata.Min(x => x.actual);
                }
                else
                {
                    bestAchivedEverMin = groupresultdata.Max(x => x.bestAchievedEver);
                    bestAchivedForSinglePeriod = groupresultdata.Max(x => x.actual);
                }

                var sqldirection = sqlKpiDataList.Where(x => x.kpiCode == request.kpiCode).Select(x => x.direction).FirstOrDefault();
                var targetMax = sqlKpiDataList.Where(x => x.kpiCode == request.kpiCode).Select(x => x.overallTargetMax).FirstOrDefault();
                foreach (var itemData in groupresultdata)
                {

                  
                   
                    itemData.direction = sqldirection!;
                    itemData.target = (int)(targetMax?? 0);
                    itemData.bestAchievedEverMin = bestAchivedEverMin;
                    itemData.bestAchievedForSinglePeriod = bestAchivedForSinglePeriod;

                }

            
            return groupresultdata!;
        }
