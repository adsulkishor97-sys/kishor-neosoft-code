public async Task<List<PlantBenchmarkGroupedData>> GetPlantBenchmark(GetPlantBenchmarkRequest request)
{
    List<PlantBenchmarkGroupedData> groupData = new List<PlantBenchmarkGroupedData>();
    //convert string to datetime
    var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
    var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);


    //convert format as yyyymm 202508
    var formatedStartdate = startDateTime.ToString("yyyyMM");
    var formatedEnddate = endDateTime.ToString("yyyyMM");


    //convert format as yyyy-mm-dd 2025-08-01
    var formatedStartdateyymmdd = startDateTime.ToString("yyyy-MM-dd");
    var formatedEnddateyymmdd = endDateTime.ToString("yyyy-MM-dd");




    PlantBenchmarkQuery plantbenchmarkquery = new PlantBenchmarkQuery();

    List<AffiliateBenchmark> jsonFileGlobalResponseList = plantbenchmarkquery.plantDataList.ToList();
    var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
    var kpiinfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);
    string quatedpmcode = "";

    // get plantIds based on affilateId
    var plantIds = request.plantId?.Split(',').Select(int.Parse).ToList();
   
    var formatPlantIds = string.Join(",", plantIds!);

    var templateIdlist = "";
    if (request.assetClassId!=0) 
    {
       templateIdlist = request.assetClassId.ToString();
    }
       

    foreach (var item in jsonFileGlobalResponseList.Where(a => a.kpiCode == request.kpiCode))
    {
        var parameters = new PlantQueryParameters
        {
            OveralN = item.bestAchievedEver ?? string.Empty,
            FormattedStartDate = formatedStartdate,
            FormattedEndDate = formatedEnddate,
            FormattedStartDateYyyyMmDd = formatedStartdateyymmdd,
            FormattedEndDateYyyyMmDd = formatedEnddateyymmdd,
            AffiliateRequest = string.Empty,
            Request = request,
            QuotedPmCode = quatedpmcode,
            PlantIds = formatPlantIds,
            TemplateIdList = templateIdlist
        };
        var queryBestAchievedEver = ReplacePlantQuery(parameters);
        var bestAchievedResult = await _benchMarkRepository.ExecuteBigDataQuery<PlantBenchmarkGroupedData>(queryBestAchievedEver, reader => new PlantBenchmarkGroupedData
        {
            plantId = reader["plant_id"] != DBNull.Value ? Convert.ToInt32(reader["plant_id"]) : 0,
            bestAchievedEver = reader["best_achieved"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved"]) : 0,
            absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
            actual = reader["actual"] != DBNull.Value ? Convert.ToDecimal(reader["actual"]) : 0

        }) ?? new List<PlantBenchmarkGroupedData>();



        if(bestAchievedResult==null || bestAchievedResult.Count == 0)
        {
            return new List<PlantBenchmarkGroupedData>
            {
                new PlantBenchmarkGroupedData
                {
                    plantId=0,
                    bestAchievedEver=0,
                    absolute=0,
                    actual=0
                }
            };
        }

        groupData = bestAchievedResult;
    }

        decimal bestAchivedEverMin;
        decimal bestAchivedForSinglePeriod;


        if (kpiinfo?.direction == 1)
        {
            bestAchivedEverMin = groupData.Min(x => x.bestAchievedEver);
            bestAchivedForSinglePeriod = groupData.Min(x => x.actual);
        }
        else
        {
            bestAchivedEverMin = groupData.Max(x => x.bestAchievedEver);
            bestAchivedForSinglePeriod = groupData.Max(x => x.actual);
        }

        long? sqltarget = sqlKpiDataList.Where(x => x.kpiCode == request.kpiCode).Select(x => x.overallTargetMax).FirstOrDefault();
    int? sqldirection = sqlKpiDataList.Where(x => x.kpiCode == request.kpiCode).Select(x => x.direction).FirstOrDefault();



    foreach (var itemData in groupData)
        {

            itemData.bestAchievedEverMin = bestAchivedEverMin;
            itemData.bestAchievedForSinglePeriod = bestAchivedForSinglePeriod;
            itemData.target = sqltarget;
            itemData.direction = sqldirection;

        }
             return groupData!;


}
