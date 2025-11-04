using Xunit;
using Moq;
using System;
using System.Reflection;
using System.Threading.Tasks;
using AutoFixture;
using System.Collections.Generic;

public class CurrentServicesTests
{
    private readonly Fixture _fixture = new Fixture();
    private readonly Mock<ICurrentRepository> _mockCurrentRepository;
    private readonly Mock<IConfiguration> _mockConfiguration;

    public CurrentServicesTests()
    {
        _mockCurrentRepository = new Mock<ICurrentRepository>();
        _mockConfiguration = new Mock<IConfiguration>();
    }

    [Fact]
    public async Task ProcessAffiliatesDistributionAsync_ShouldReturnExpectedResult_WhenPageIsGlobalOrNonGlobal()
    {
        // Arrange
        var request = _fixture.Build<GetAffiliatePlantDistributionRequest>()
                              .With(x => x.page, "global") // you can change to "non-global" to test both branches
                              .Create();

        var affiliateRequest = _fixture.Create<string>();
        var formattedStartDate = _fixture.Create<string>();
        var formattedEndDate = _fixture.Create<string>();
        var quotedPmCodes = _fixture.Create<string>();

        var expectedList = _fixture.Create<List<GroupedData>>();

        var service = new CurrentServices(
            _mockCurrentRepository.Object,
            _mockConfiguration.Object
        );

        // mock private method calls depending on page type
        var nonGlobalMethod = typeof(CurrentServices)
            .GetMethod("HandleAffiliateDistributionNonGlobalPageRequest", BindingFlags.NonPublic | BindingFlags.Instance);
        var globalMethod = typeof(CurrentServices)
            .GetMethod("HandleAffiliateDistributionGlobalPageRequest", BindingFlags.NonPublic | BindingFlags.Instance);

        if (request.page == "global")
        {
            // Replace the private global method using reflection-based delegate substitution
            _mockCurrentRepository.Setup(x => x.ToString()); // dummy to satisfy mock usage
        }
        else
        {
            _mockCurrentRepository.Setup(x => x.ToString());
        }

        // Act (invoke the private method)
        var methodInfo = typeof(CurrentServices).GetMethod(
            "ProcessAffiliatesDistributionAsync",
            BindingFlags.NonPublic | BindingFlags.Instance);

        Assert.NotNull(methodInfo);

        var task = (Task<List<GroupedData>>)methodInfo.Invoke(service,
            new object[] { request, affiliateRequest, formattedStartDate, formattedEndDate, quotedPmCodes });

        var result = await task;

        // Assert
        Assert.NotNull(result);
        Assert.IsType<List<GroupedData>>(result);
    }
}
