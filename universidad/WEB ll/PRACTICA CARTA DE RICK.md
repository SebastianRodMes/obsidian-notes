  <div class="card">

        <img src="Rick.png" alt="Rick Sanchez">

        <h2>Rick Sanchez</h2>

        <p>Especie: humano</p>

        <p>Estado: Vivo</p>

  

    </div>
   * {

  margin: 0;

  padding: 0;

  box-sizing: border-box;

}

  

.card{

    width: 600px;

    background-color: aqua;

  display: flex;

  justify-content: center;

  align-items: center;

  gap: 20px;

  flex-direction: column; CON ESTO SE ORDENA EN VERTICAL HACIA ABAJO

  

}
- `flex-direction: column` → apila los elementos **de arriba a abajo**.
    
- `justify-content: center` → centra los hijos **verticalmente**.
    
- `align-items: center` → centra los hijos **horizontalmente**.