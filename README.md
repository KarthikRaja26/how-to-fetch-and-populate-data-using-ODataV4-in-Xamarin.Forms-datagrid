# how-to-fetch-and-populate-data-using-ODataV4-in-Xamarin.Forms-datagrid

## About the sample
This example illustrates how to fetch and popuplate data using ODataV4 in Xmarin.Forms SfDataGrid.

Simple.OData.Client is a cross-platform client library through which you can fetch and update data online. In, the below sample we fetched the data from online and loaded those data to SfDataGrid by using Simple.OData.Client library.

```C#
public class ResultsPage : ContentPage
{
        IEnumerable<Package> packages;
        SfDataGrid dataGrid;
        ActivityIndicator activityIndicator;
        public ResultsPage()
        {
            Title = "Search Results";

            NavigationPage.SetHasNavigationBar(this, true);

            var stackLayout = new StackLayout() { VerticalOptions = LayoutOptions.FillAndExpand };

            if (Device.OS == TargetPlatform.WinPhone)
            {
                // WinPhone doesn't have the title showing
                stackLayout.Children.Add(new Label { Text = Title, Font = Font.SystemFontOfSize(50) });
            }

            var searchButton = new Button() { Text = "Get Data" };
            searchButton.Clicked += async (sender, e) =>
            {
                try
                {
                    activityIndicator.IsVisible = true;
                    packages = await GetPackages();
                    if (packages != null)
                        SetSource();
                    activityIndicator.IsVisible = false;
                }
                catch(Exception)
                {
                    await DisplayAlert("Error","Connect to the internet and try again..!","OK");
                    activityIndicator.IsVisible = false;
                }
            };

            var grid = new Grid();
            activityIndicator = new ActivityIndicator();
            activityIndicator.HeightRequest = 100;
            activityIndicator.HorizontalOptions = LayoutOptions.Center;
            activityIndicator.VerticalOptions = LayoutOptions.Center;
            activityIndicator.IsEnabled = true;
            activityIndicator.IsRunning = true;
            activityIndicator.IsVisible = false;

            dataGrid = new SfDataGrid();
            dataGrid.ColumnSizer = ColumnSizer.Auto;
            dataGrid.GridTapped += (sender, e) =>
            {
                var package = (PackageViewModel)e.RowData;
                var detailsPage = new DetailsPage();
                detailsPage.BindingContext = package;
                Navigation.PushAsync(detailsPage);
            };

            grid.Children.Add(activityIndicator);
            grid.Children.Add(dataGrid);
            stackLayout.Children.Add(searchButton);
            stackLayout.Children.Add(grid);

            this.Content = stackLayout;
        }

        private void SetSource()
        {
            var results = packages.Select(x => new PackageViewModel(x));
            dataGrid.ItemsSource = results;
        }

        private async Task<IEnumerable<Package>>  GetPackages()
        {
            var odataClient = new ODataClient("https://nuget.org/api/v1");
            var command = odataClient
                .For<Package>("Packages")
                .OrderByDescending(x => x.DownloadCount)
                .Top(2);

            command.OrderBy(x => x.Id);
            command.Filter(x => x.Title.Contains("Xamarin") && x.IsLatestVersion);
            command.Select(x => new { x.Id, x.Title, x.Version, x.LastUpdated, x.DownloadCount, x.VersionDownloadCount, x.PackageSize, x.Authors, x.Dependencies });

            return await command.FindEntriesAsync();
        }
}
```

## <a name="requirements-to-run-the-demo"></a>Requirements to run the demo ##

* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/) or [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/).
* Xamarin add-ons for Visual Studio (available via the Visual Studio installer).

## <a name="troubleshooting"></a>Troubleshooting ##
### Path too long exception
If you are facing path too long exception when building this example project, close Visual Studio and rename the repository to short and build the project.
