**Backend**

In de back-end voer ik tests uit om ervoor te zorgen dat de kwaliteit en betrouwbaarheid gecontroleerd en aangetoond kan worden. Dit is belangrijk 
om te verifiëren of alles naar verwachting werkt. Maar ook om dit in een professionele setting te kunnen bewijzen.

Hier leg ik een van de tests uit om te laten zien hoe het in zijn werk gaat. Deze test voert een Get all operatie uit waarbij een lijst van TaskItems
moet worden teruggegeven.

        public async Task Get_ReturnsListOfTasks()
        {
            // Arrange
            var taskServiceMock = new Mock<ITaskService>();
            var expectedTasks = new List<TaskItem> { new TaskItem { Id = "1", TaskName = "Task 1" }, new TaskItem { Id = "2", TaskName = "Task 2" } };
            taskServiceMock.Setup(service => service.GetAsync()).ReturnsAsync(expectedTasks);

            var controller = new TasksController(taskServiceMock.Object);

            // Act
            var result = await controller.Get();

            // Assert
            var actionResult = Assert.IsType<ActionResult<List<TaskItem>>>(result);
            var model = Assert.IsType<List<TaskItem>>(actionResult.Value);
            Assert.Equal(expectedTasks, model);
        }

**Structuur**

- **Arrange**
In de arrange sectie worden de noodzakelijke variabelen en objecten aangemaakt
die nodig zijn om informatie uit de get all te halen. Ook wordt hier een mock aangemaakt van de database. Hierdoor kan de database getest worden
zonder daadwerkelijke data te wijzigen.

- **Act** Vervolgens wordt in de Act sectie de Get all controller call uitgevoerd. 
- **Assert** In de Assert sectie wordt het resultaat vergelegen met de lijst
die in de Arrange sectie is aangemaakt om te zien of deze gelijk zijn.

Voor deze query zijn ook testen gemaakt om te controleren wat er gebeurd wanneer de juiste informatie niet beschikbaar is. In de volgende test wordt gecontroleerd of de Get all juist een NotFound returned wanneer de lijst leeg is:

        public async Task Get_NoTasks_ReturnsNotFound()
        {
            // Arrange
            var taskServiceMock = new Mock<ITaskService>();
            taskServiceMock.Setup(service => service.GetAsync()).ReturnsAsync(new List<TaskItem>());

            var controller = new TasksController(taskServiceMock.Object);

            // Act
            var result = await controller.Get();

            // Assert
            var notFoundResult = Assert.IsType<NotFoundResult>(result.Result);
            Assert.Equal(404, notFoundResult.StatusCode);
        }

**Auth0**

Om de veiligheid van de app te verbeteren is Auth0 geïmplementeerd.
Dit heb ik gedaan door op de site van Auth0 een account aan te maken waarbij een lijst van accounts kan worden toegevoegd. Vervolgens wordt de gebruiker bij het inloggen naar de Auth0 site word gestuurd waar de app gebruikersinformatie terugkrijgt zoals een emailadres en profielfoto:

 ![Alt text](../Images/Auth0front-end.png)

 dit is mogelijk door in index.js de Auth0 provider toe te voegen waarna de login en loguit functionaliteiten kunnen worden toegevoegd:

 const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
```
  <Auth0Provider
      domain="dev-l6zgzkm7kvwifn0f.us.auth0.com"
      clientId="o4l9jBFG3zhKJU2zXyRu7dcpDfa88q0e"
      authorizationParams={{
        redirect_uri: window.location.origin
      }}
    >
      <App />
    </Auth0Provider>
```
