**Back-end**

Voor de back-end heb ik ASP.NET gekozen. In eerste instantie heb ik hiervoor gekozen met het oog op een Azure deployment. Omdat deze beide door Microsoft worden onderhouden en goed met elkaar werken. Later ben ik echter meer naar Docker gaan kijken.
Maar er zijn ook andere redenen waarom ASP.NET een goede keuze is. Zo zijn websites vaak sneller en efficiënter dan wanneer deze bijvoorbeeld met PHP worden gemaakt.

**MongoDB**
Eerder in het semester heb ik gebruik gemaakt van een lokale MySQL server. Echter wilde ik graag gebruik maken van een gehoste server, zodat ik deze altijd makkelijk kon benaderen zonder deze eerst lokaal aan te zetten. En dit kan later ook gebruikt worden als ik mijn back-end wil deployen.

Omdat ik het ook lastig vond om een gratis mysql server te vinden, ben ik verder gaan kijken, en ben ik terechtgekomen bij MongoDB. Dit is een NoSQL server waardoor de model niet in de database gedefiniëerd hoeft te worden. Dit is heel handig als je de models later nog wilt aanpassen. Dit kan dan zonder problemen.

Omdat MongoDB een hele andere opzet heeft dan MYSQL, heb ik hier wel mijn hele back-end voor moeten veranderen.

**Service layer**

Voor de Controller heb ik een service klasse aangemaakt die communiceert met de database:

    public class TasksService : ITaskService
    {
        private readonly IMongoCollection<TaskItem> _taskItemsCollection;

        public TasksService(
            IOptions<TaskDatabaseSettings> taskDatabaseSettings)
        {
            var mongoClient = new MongoClient(
                taskDatabaseSettings.Value.ConnectionString);

            var mongoDatabase = mongoClient.GetDatabase(
                taskDatabaseSettings.Value.DatabaseName);

            _taskItemsCollection = mongoDatabase.GetCollection<TaskItem>(
                taskDatabaseSettings.Value.TasksCollectionName);
        }

Voor deze service is ook een interface aangemaakt voor dependency injection:

using agenda_backend.Models;

    namespace agenda_backend.Interfaces
    {
        public interface ITaskService
        {
            Task<List<TaskItem>> GetAsync();
            Task<TaskItem?> GetAsync(string id);
            Task CreateAsync(TaskItem newTaskItem);
            Task UpdateAsync(string id, TaskItem updatedTaskItem);
            Task RemoveAsync(string id);
        }
    }

Hierdoor blijft de controller beter gescheiden van de service, waardoor beide makkelijker onafhankelijk van elkaar aangepast kunnen worden als dit nodig is.

    builder.Services.Configure<TaskDatabaseSettings>(
        builder.Configuration.GetSection("TaskDatabase"));

    builder.Services.AddSingleton<ITaskService,TasksService>();

Er is ook een aparte TaskDatabase klasse gemaakt die de connectionstring, database en collections ophaald voor de rest van de applicatie.