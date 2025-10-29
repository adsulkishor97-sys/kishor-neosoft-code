public void GetElementPerformanceKPIDetails(List<PIServer> piServers)
{
    try
    {
        var elementIds = _elementPerformanceKpiDetailsRepository.GetElementIds();

        foreach (var elementId in elementIds)
        {
            ProcessElementPerformance(elementId);
        }
    }
    catch (Exception)
    {
        // will implement DBLogger
    }
}
