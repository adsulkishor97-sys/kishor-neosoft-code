var tasks = FileGlobalResponseList
    .AsParallel()
    .Where(a => a.kpiCode == request.kpiCode)
    .Select(async item =>
    {
        // ✅ Build the parameter object
        var parameters = new AssetQueryParameters
        {
            OveralN = item.bestAchievedEver ?? string.Empty,
            FormattedStartDate = formatedStartdate,
            FormattedEndDate = formatedEnddate,
            FormattedStartDateYyyyMmDd = formatedStartdateyymmdd,
            FormattedEndDateYyyyMmDd = formatedEnddateyymmdd,
            AffiliateRequest = string.Empty,
            Request = request,
            QuotedPmCode = quatedpmcode,
            SapIds = formatSapIds
        };

        // ✅ Call the new refactored ReplaceAssetQuery method
        var queryBestAchievedEver = ReplaceAssetQuery(parameters);

        // ✅ Execute query
        var bestAchievedResult = await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(
            queryBestAchievedEver,
            reader => new AssetBenchmarkGroupedData
            {
                manufacturer = reader["manufacturer"] != DBNull.Value ? Convert.ToString(reader["manufacturer"]) : "",
                modelNumber = reader["model_number"] != DBNull.Value ? Convert.ToString(reader["model_number"]) : "",
                bestAchievedEver = reader["best_achieved_ever"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved_ever"]) : 0,
                absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
                actual = reader["actual"] != DBNull.Value ? Convert.ToInt32(reader["actual"]) : 0,
                sapId = reader["sap_id"] != DBNull.Value ? Convert.ToString(reader["sap_id"])!.TrimStart('0') : ""
            }
        ) ?? new List<AssetBenchmarkGroupedData>();

        return bestAchievedResult;
    });
