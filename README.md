public async Task<GetTotalCostOfOwnershipResponse> GetTotalCostOfOwnershipByPlantIdAsync(GetTotalCostOfOwnershipRequestByplantId request)
{
    if (request == null)
        throw new ArgumentNullException(nameof(request), "Request cannot be null");

    // Step 1: Format dates
    var (formattedStartDate, formattedEndDate, startMonth, endMonth) = FormatTcoDates(request.startDate, request.endDate);

    // Step 2: Get raw TCO data
    var rawResponses = await GetRawTcoDataAsync(request.plantId, formattedStartDate, formattedEndDate, startMonth, endMonth);

    // Step 3: Aggregate and compute totals & percentages
    var finalResponse = ComputeTcoTotals(rawResponses);

    return finalResponse;
}

#region Private Helpers

private static (string formattedStartDate, string formattedEndDate, string startMonth, string endMonth) FormatTcoDates(string? startDate, string? endDate)
{
    var startDateTime = !string.IsNullOrEmpty(startDate) ? DateTime.Parse(startDate, CultureInfo.InvariantCulture) : DateTime.Now;
    var endDateTime = !string.IsNullOrEmpty(endDate) ? DateTime.Parse(endDate, CultureInfo.InvariantCulture) : DateTime.Now;

    return (
        startDateTime.ToString("yyyy-MM-dd"),
        endDateTime.ToString("yyyy-MM-dd"),
        startDateTime.ToString("yyyyMM"),
        endDateTime.ToString("yyyyMM")
    );
}

private async Task<List<GetTotalCostOfOwnershipResponse>> GetRawTcoDataAsync(string plantId, string formattedStartDate, string formattedEndDate, string startMonth, string endMonth)
{
    var responseList = new List<GetTotalCostOfOwnershipResponse>();
    var affiliateTco = new AffiliateTcoByPlantId();

    foreach (var item in affiliateTco.totalCostOfOwnerships)
    {
        var query = ReplaceTcoQuery(item.query!, plantId, formattedStartDate, formattedEndDate, startMonth, endMonth);

        var result = await _currentRepository.ExecuteBigDataQuery_New<GetTotalCostOfOwnershipResponse>(query, reader => new GetTotalCostOfOwnershipResponse
        {
            maintenance = reader["maintenance"] != DBNull.Value ? Convert.ToDecimal(reader["maintenance"]) : 0,
            disposal = reader["disposal"] != DBNull.Value ? Convert.ToDecimal(reader["disposal"]) : 0,
            acquisition = reader["acquisition"] != DBNull.Value ? Convert.ToDecimal(reader["acquisition"]) : 0,
            operation = reader["operation"] != DBNull.Value ? Convert.ToDecimal(reader["operation"]) : 0,
            productionLoss = reader["production_loss"] != DBNull.Value ? Convert.ToDecimal(reader["production_loss"]) : 0
        }) ?? new List<GetTotalCostOfOwnershipResponse>();

        responseList.AddRange(result);
    }

    return responseList;
}

private static string ReplaceTcoQuery(string query, string plantId, string formattedStartDate, string formattedEndDate, string startMonth, string endMonth)
{
    return query
        .Replace("plantId", plantId)
        .Replace("startDate", formattedStartDate)
        .Replace("endDate", formattedEndDate)
        .Replace("formatedStartdate", startMonth)
        .Replace("formatedEnddate", endMonth);
}

private static GetTotalCostOfOwnershipResponse ComputeTcoTotals(List<GetTotalCostOfOwnershipResponse> responses)
{
    var first = responses.FirstOrDefault() ?? new GetTotalCostOfOwnershipResponse();

    var finalResponse = new GetTotalCostOfOwnershipResponse
    {
        maintenance = Math.Round(first.maintenance, 0),
        disposal = Math.Round(first.disposal, 0),
        acquisition = Math.Round(first.acquisition, 0),
        operation = Math.Round(first.operation, 0),
        productionLoss = Math.Round(first.productionLoss, 0)
    };

    finalResponse.total = finalResponse.maintenance + finalResponse.disposal + finalResponse.acquisition + finalResponse.operation + finalResponse.productionLoss;
    finalResponse.totalAssetOperation = finalResponse.productionLoss + finalResponse.operation + finalResponse.maintenance;

    if (finalResponse.total > 0)
    {
        finalResponse.maintenancePercent = Math.Round((finalResponse.maintenance / finalResponse.total) * 100, 2);
        finalResponse.disposalPercent = Math.Round((finalResponse.disposal / finalResponse.total) * 100, 2);
        finalResponse.acquisitionPercent = Math.Round((finalResponse.acquisition / finalResponse.total) * 100, 2);
        finalResponse.operationPercent = Math.Round((finalResponse.operation / finalResponse.total) * 100, 2);
        finalResponse.productionLossPercent = Math.Round((finalResponse.productionLoss / finalResponse.total) * 100, 2);
    }

    return finalResponse;
}

#endregion
