**Front-end**

Voor de Front-end heb ik React gebruikt.
Dit is niet alleen voor het gebruik van Javascript, maar ook voor het gebruik van components. Hiermee kun je je applicatie opdelen in kleine stukjes, waardoor het veel makkelijker is om dingen toe te voegen en aan te passen.

Ik zal hier een voorbeeld geven van hoe ik deze components gebruik in de front-end. Zo heb ik een Tasklist component die een lijst van Tasks ophaalt. Deze tasklist heeft ook de functies om tasks te verwijderen.

    useEffect(() => {
        fetchTasks()
    }, [])

Door de brackets aan het eind van de useEffect te zetten, zorg ik ervoor dat deze alleen wordt geüpdate wanneer de state veranderd.

    const addTask = async (taskName) => {
      try {
        const response = await fetch("https://localhost:7140/api/Tasks", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            TaskName: taskName,
          }),
        });
  
        if (response.ok) {
          const data = await response.json();
          setTaskList([...taskList, data]);
          setNewTaskName("");
        } else {
          console.error("Failed to add task");
        }
      } catch (error) {
        console.error("Error adding task", error);
      }
    };

    const deleteTask = async (taskId) => {
      try {
        const response = await fetch(`https://localhost:7140/api/Tasks/${taskId}`, {
          method: "DELETE",
        });
  
        if (response.ok) {
          setTaskList(taskList.filter((item) => item.id !== taskId));
        } else {
          console.error("Failed to delete task");
        }
      } catch (error) {
        console.error("Error deleting task", error);
      }
    };

Nu heb ik ook een aparte Task component, waarin ik de delete functie van de TaskList component kan doorgeven:

        {taskList.map((item, index) => (
            <p key={index}>
                <Task taskName={item.taskName} taskId={item.id} onDelete={deleteTask}/>
            </p>

Nu kan ik vanuit de individuele tasks de deleteknop gebruiken om deze in de TaskList component te verwijderen:

    <>
      <p>{task.name}</p>
      <button onClick={updateIsCompleted}>Complete</button>
      <button onClick={handleDeleteClick}>Delete</button>
    </>

 Omdat de Tasklist ook direct wordt geüpdate in de delete functie wordt de useEffect aangeroepen en wordt de up to date lijst weergeven.

 Met deze functionaliteiten wordt de volgende lijst bewerkt:

 ![Alt text](../Images/Auth0front-end.png)