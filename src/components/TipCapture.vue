<template>
    <div>
      <div class="total-line">
        <p class="efectivo-class">Efectivo en Caja</p>
        <p class="total-text">$ {{ totalInCash.toFixed(2) }}</p>
        <p class="efectivo-class2"> Pago de Propinas</p>
        <div class="line"></div>
      </div>
  
      <div class="tip-capture-container">
        <div class="text-container">
          <label for="tipAmount" class="tip-label">Total de Propinas</label>
          <div class="submitted-tip" @click="toggleEdit">
            <div class="submitted-tip-box">
              <p>$ {{ submittedTip !== null ? submittedTip.toFixed(2) : '0.00' }}</p>
              <span class="edit-icon" v-if="!isEditing">
                <i class="fas fa-edit"></i>
              </span>
              <img
                src="https://w7.pngwing.com/pngs/740/67/png-transparent-computer-icons-icon-design-edit-angle-computer-data-thumbnail.png"
                alt="Edit Icon"
                class="edit-image"
                v-if="!isEditing"
              />
            </div>
          </div>
        </div>
        <div class="division-container">
          <label for="divideNumber" class="divide-label">¿Entre cuántos quieres dividir las</label>
          <p class="divide-label2">Propinas?</p>
          <br><input v-model="divideNumber" type="text" id="divideNumber" class="divide-input" @keyup.enter="handleDivision" />
          <div class="division-result">
            $ {{ divisionResult !== null ? divisionResult.toFixed(2) + ' (' + divisionResultMessage + ')' : '0.00' }} x personas
          </div>
        </div>
        <div class="payment-method-container">
          <label for="paymentMethod" class="payment-label">Elige el Método de Pago</label>
          <img src="@/assets/cartera.png" alt="Icono de Cartera" class="payment-icon" />
          <div class="payment-options">
            <div
              class="payment-card cash-card"
              :class="{ 'isCashSelected': isCashSelected }"
              style="left: 0;"
              @click="togglePaymentSelection('cash')"
            >
              <img
                src="@/assets/efectivo.png"
                alt="Efectivo Icon"
                class="payment-icon2"
                :style="{ filter: isCashSelected ? 'invert(1)' : 'invert(0)' }"
              />
              <label
                for="cash"
                :style="{ color: isCashSelected ? '#fff' : '#252424' }"
              >
                Efectivo
              </label>
            </div>
            <div
              class="payment-card card-card"
              :class="{ 'isCardSelected': isCardSelected }"
              style="left: 120px;"
              @click="togglePaymentSelection('card')"
            >
              <img
                src="@/assets/tarjeta.png"
                alt="Tarjeta Icon"
                class="payment-icon2"
                :style="{ filter: isCardSelected ? 'invert(1)' : 'invert(0)' }"
              />
              <label
                for="card"
                :style="{ color: isCardSelected ? '#fff' : '#252424' }"
              >
                Tarjeta
              </label>
            </div>
            <div
              class="payment-card card5-card"
              :class="{ 'isCard5Selected': isCard5Selected }"
              style="left: 240px;"
              @click="togglePaymentSelection('card5')"
            >
              <img
                src="@/assets/tarjeta.png"
                alt="Tarjeta 5 Icon"
                class="payment-icon2"
                :style="{ filter: isCard5Selected ? 'invert(1)' : 'invert(0)' }"
              />
              <label
                for="card5"
                :style="{ color: isCard5Selected ? '#fff' : '#252424' }"
              >
                Tarjeta 5
              </label>
            </div>
          </div>
        </div>
        <div class="calculator-container">
          <div class="calculator">
            <div class="calculator-screen">
              <span>{{ currentInput }}</span>
              <button class="clear-button" @click="handleButtonClick('C')">C</button>
            </div>
            <div class="calculator-buttons">
              <button v-for="button in calculatorButtons" :key="button" @click="handleButtonClick(button)">
                {{ button }}
              </button>
            </div>
          </div>
          <div v-if="isEditing" class="overlay">
            <div class="edit-dialog">
              <label for="editedTip" class="edit-label">Editar Monto:</label>
              <input v-model="editedTip" type="text" id="editedTip" class="edit-input" />
              <button @click="handleEditConfirm">Confirmar</button>
              <button @click="toggleEdit">Cancelar</button>
            </div>
          </div>
        </div>
      </div>
  
      <div class="pagos-container">
        <h2>Pagos Realizados</h2>
        <div v-for="(pago, index) in pagosRealizados" :key="index" class="pago-message">
          <p>{{ pago.metodoPago }}: $ {{ pago.monto.toFixed(2) }}</p>
          <img :src="pago.metodoPagoIcono" :alt="pago.metodoPago" class="pago-icon" />
          <button class="delete-button" @click="eliminarPago(index)">X</button>

        </div>
      </div>
  
      <div v-if="pagoRealizado" class="pago-message">
        <p>{{ metodoPagoSeleccionado }}: $ {{ montoPagado.toFixed(2) }}</p>
        <img :src="metodoPagoIcono" :alt="metodoPagoSeleccionado" class="pago-icon" />
      </div>
  
      <!-- Sección de resumen -->
      <div class="total-summary">
  <p>Total Pagado: $ {{ totalPagado.toFixed(2) }}</p>
  <p>Restante por Pagar: $ {{ (totalPropinas - totalPagado).toFixed(2) }}</p>
</div>
<div class="propinas-container">
  <button :class="{ 'propinas-button': true, 'completed': restantePorPagar <= 0 }" @click="pagarPropinas">
    {{ restantePorPagar <= 0 ? 'Pagar $ ' + totalPropinas : 'Pagar Propinas' }}
  </button>
</div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        tipAmount: 0,
        currentInput: '0',
        calculatorButtons: ['1', '2', '3', '4', '5', '6', '7', '8', '9', '0', '00', '✅'],
        submittedTip: null,
        editedTip: null,
        isEditing: false,
        totalInCash: 0,
        divideNumber: 1,
        divisionResult: null,
        isCashSelected: false,
        isCardSelected: false,
        isCard5Selected: false,
        pagosRealizados: [],
        totalPagado: 0,
        totalPropinas: 0,
      };
    },
    computed: {
      formattedTipAmount() {
        return parseFloat(this.currentInput).toFixed(2);
      },
      divisionResultMessage() {
        return `${this.divideNumber} `;
      },

      // Calcula el restante por pagar utilizando una propiedad computada
      restantePorPagar() {
    return this.totalPropinas - this.totalPagado;
  },
      
    },
    methods: {
      handleButtonClick(button) {
        if (button === 'C') {
          this.currentInput = this.currentInput.slice(0, -1);
          if (this.currentInput === '') {
            this.currentInput = '0';
          }
        } else if (button === '✅') {
          this.mostrarMensajePago();
        } else {
          this.currentInput =
            this.currentInput === '0' ? button : this.currentInput + button;
        }
      },
  
      mostrarMensajePago() {
        let nuevoPago = {};
        if (this.isCashSelected) {
          nuevoPago.metodoPago = 'Efectivo';
          nuevoPago.metodoPagoIcono = require('@/assets/efectivo.png');
        } else if (this.isCardSelected) {
          nuevoPago.metodoPago = 'Tarjeta';
          nuevoPago.metodoPagoIcono = require('@/assets/tarjeta.png');
        } else if (this.isCard5Selected) {
          nuevoPago.metodoPago = 'Tarjeta 5';
          nuevoPago.metodoPagoIcono = require('@/assets/tarjeta.png');
        }
  
        nuevoPago.monto = parseFloat(this.currentInput);
        this.pagosRealizados.push(nuevoPago);
  
        // Actualizar total pagado
        this.totalPagado += nuevoPago.monto;
  
        // Actualizar total propinas
        this.totalPropinas += this.submittedTip !== null ? this.submittedTip : 0;
      },
  
      toggleEdit() {
        this.isEditing = !this.isEditing;
      },
  
      handleEditConfirm() {
        this.isEditing = false;
        if (this.editedTip !== null) {
          this.submittedTip = parseFloat(this.editedTip);
        }
      },
  
      handleDivision() {
        if (this.submittedTip !== null && this.divideNumber !== 0) {
          this.divisionResult = this.submittedTip / this.divideNumber;
          this.currentInput = this.divisionResult.toFixed(2);
        } else {
          alert("No se puede dividir entre 0. Por favor, ingresa un número válido.");
        }
      },
  
      togglePaymentSelection(type) {
        if (type === 'cash') {
          this.isCashSelected = true;
          this.isCardSelected = false;
          this.isCard5Selected = false;
        } else if (type === 'card') {
          this.isCashSelected = false;
          this.isCardSelected = true;
          this.isCard5Selected = false;
        } else if (type === 'card5') {
          this.isCashSelected = false;
          this.isCardSelected = false;
          this.isCard5Selected = true;
        }
      },
  
      resetPaymentSelection() {
        this.isCashSelected = false;
        this.isCardSelected = false;
        this.isCard5Selected = false;
      },
      eliminarPago(index) {
      // Restar el monto eliminado del total pagado
      this.totalPagado -= this.pagosRealizados[index].monto;

      // Eliminar el pago del array de pagos realizados
      this.pagosRealizados.splice(index, 1);
    },
    pagarPropinas() {
      if (this.restantePorPagar <= 0) {
        // Lógica para procesar el pago de propinas
        // Actualiza el estado de propinasCompletas y realiza otras acciones necesarias
        this.propinasCompletas = true;
      }
    },
  },
  };
  </script>
  
  <style scoped>
  .tip-capture-container {
    display: flex;
    align-items: center;
    margin: 100px;
  }
  
  .text-container {
    margin-right: 280px;
    margin-top: -400px;
  }
  
  .tip-label {
    color: #ff0000;
    font-weight: bold;
    font-family: 'Segoe UI', sans-serif;
    font-size: 14px;
    margin-top: -2000px;
  }
  
  .calculator-container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  .calculator {
    border: 1px solid #ddd;
    border-radius: 10px; /* Aumenta el radio de los bordes para un aspecto más suave */
    margin: 20px 0; /* Aumenta el margen superior e inferior */
    padding: 20px;
    width: 240px; /* Aumenta el ancho de la calculadora */
    height: 300px; /* Aumenta la altura de la calculadora */
    background-color: #f7f7f7; /* Cambia el color de fondo para una apariencia más clara */
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Agrega una sombra para un efecto 3D */
    margin-top: -95px;
    margin-left: -120px;
  }

  .calculator-screen {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border: 1px solid #ddd;
    border-radius: 5px;
    margin-bottom: 10px;
    padding: 10px; /* Reduzca el espacio interno para un diseño más limpio */
    text-align: right;
    width: 100%;
    box-sizing: border-box;
    background-color: #f0f0f0; /* Cambia el color de fondo de la pantalla */
  }

  .calculator-buttons {
    display: grid;
    grid-template-columns: repeat(4, 1fr); /* Cambia a una cuadrícula de 4 columnas */
    gap: 10px;
  }

  button {
    padding: 15px;
    border: 1px solid #ddd;
    border-radius: 5px;
    cursor: pointer;
    width: 100%;
    background-color: #fff;
    color: #333;
    transition: background-color 0.3s ease; /* Agrega una transición para suavizar los cambios de color */
  }

  button:active {
    background-color: #ddd;
  }
  
  .clear-button {
    padding: 1px;
    font-size: 12px;
    background-color: #ffcccc;
    color: #ff0000;
    width: 15px;
  }
  
  .calculator-buttons {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 5px;
  }
  
  .submitted-tip-box {
    background-color: #ffcccc;
    padding: 1px;
    border: 0px solid #ff0000;
    border-radius: 5px;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    font-size: 25px;
    color: #ff0000;
    height: 55px;
    width: 160px;
    padding-left: 20px;
    font-family: 'Segoe UI';
    font-weight: bold;
  }

  .submitted-tip-box p {
  margin-top: 10px; /* Ajusta el valor según sea necesario */
}
  
  .edit-icon {
    margin-top: -65px;
    cursor: pointer;
    color: #007bff;
  }
  
  .edit-image {
    width: 25px;
    margin-left: 180px;
  }
  
  .overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .edit-dialog {
    background-color: #fff;
    padding: 20px;
    border-radius: 5px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
    text-align: center;
  }
  
  .edit-label {
    display: block;
    margin-bottom: 10px;
  }
  
  .edit-input {
    width: 100%;
    padding: 10px;
    margin-bottom: 15px;
  }
  
  .total-line {
    text-align: center;
    position: relative;
    margin-bottom: 20px;
  }
  
  .efectivo-class {
    padding: 15px;
    margin-left: 1200px;
    font-size: 11px;
    margin-top: -10px;
    color: #ff0000;
    display: flex;
    font-family: 'Segoe UI';
  }
  
  .efectivo-class2 {
    margin-left: 45px;
    font-size: 20px;
    margin-top: -50px;
    display: flex;
    font-family: 'Segoe UI';
    font-weight: bold;
  }
  
  .total-text {
    margin: 0;
    font-size: 25px;
    color: #ff0000;
    font-weight: bold;
    width: 150px;
    background-color: #ffcccc;
    padding: 10px;
    padding-left: -50px;
    margin-left: 1165px;
    margin-top: -25px;
    font-family: 'Segoe UI';
  }
  
  .line {
    position: absolute;
    width: 100%;
    height: 1px;
    background-color: #000000;
    top: 50%;
    transform: translateY(-50%);
    margin-top: 55px;
  }
  
  .division-container {
    margin-top: 100px;
    
  }

  .divide-label {
    margin-left: -500px;
    color: #252424;
    font-weight: bold;
    font-family: 'Segoe UI';
    font-size: 15px;
    margin-top: 100px;
    position: relative;
    top: -45px;

    
  }

  .divide-label2 {
    margin-left: -500px;
    color: #252424;
    font-weight: bold;
    font-family: 'Segoe UI';
    font-size: 15px;
    margin-top: 100px;
    position: relative;
    top: -145px;  

    
  }

  .divide-input {
    width: 55px;
    padding: 8px;
    margin-left: -459px;
    border-radius: 10px;
    margin-top: 10px;
    position: relative;
    top: -180px;
  }

  .division-result {
    margin-top: -23px;
    font-weight: bold;
    margin-left: -365px;
    font-family: 'Segoe UI';
    color: #ff0000;
    position: relative;
    top: -190px;
  }

  .payment-method-container {
  text-align: center;
  margin-top: 200px;
}

.payment-label {
  color: #252424;
  font-weight: bold;
  font-family: 'Segoe UI';
  font-size: 15px;
  margin-left: -950px;
  
}

.payment-icon {
  width: 20px; /* Ajusta el tamaño según sea necesario */
  height: 20px; /* Ajusta el tamaño según sea necesario */
  margin-top: 10px;
  margin-left: -205px;
}

.payment-icon2 {
    width: 20px; /* Ajusta el tamaño según sea necesario */
  height: 20px; /* Ajusta el tamaño según sea necesario */
  margin-top: 10px;
}

.payment-options {
  margin: 0;
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  margin-top: 10px;
  margin-left: -500px;
}

.payment-card {
  border: 1px solid #ddd;
  border-radius: 5px; /* Añade borde circular */
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1); /* Añade sombra */
  cursor: pointer;
  margin-left: -8px;
  transition: background-color 0.3s ease;
  padding: 10px;
  background-color: #fff;
  cursor: pointer;
  width: 100px;
  height: 45px;
  cursor: pointer;
  margin: 0;
  font-family: 'Segoe UI';
}

.cash-card {
    margin-left: -8px;
    background-color: #fff; /* Color de fondo predeterminado */
    border: 1px solid #ddd;
    border-radius: 5px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    cursor: pointer;
    transition: background-color 0.3s ease;
    font-family: 'Segoe UI';
    font-size: 12px;
  }

.card-card {
    margin-left: -130px;
    background-color: #fff; /* Color de fondo predeterminado */
    border: 1px solid #ddd;
    border-radius: 5px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    cursor: pointer;
    transition: background-color 0.3s ease;
    font-family: 'Segoe UI';
    font-size: 12px;
  }

.card5-card {
    margin-left: -8px;
    background-color: #fff; /* Color de fondo predeterminado */
    border: 1px solid #ddd;
    border-radius: 5px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    cursor: pointer;
    transition: background-color 0.3s ease;
    font-family: 'Segoe UI';
    font-size: 12px;
    
  }

.payment-card:hover {
  background-color: #f0f0f0; /* Cambia el color de fondo al pasar el ratón */
  gap: 5px;
}

/* Agrega estilos para el color seleccionado */
.isCashSelected {
  background-color: #ff0000; /* Color rojo para Efectivo seleccionado */
}

.isCardSelected {
  background-color: #ff0000; /* Color rojo para Tarjeta seleccionada */
}

.isCard5Selected {
  background-color: #ff0000; /* Color rojo para Tarjeta 5 seleccionada */
}

.pagos-container {
    text-align: center;
    margin-top: -470px;
    margin-left: 500px;
    font-family: 'Segoe UI';
    font-size: 10px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .pago-message {
    margin-top: 10px;
    text-align: center;
    background-color: #ffffff; /* Cambia este color según tus preferencias */
    padding: 10px; /* Ajusta el espacio interno según sea necesario */
    border-radius: 10px; /* Controla el radio de los bordes */
    display: flex; /* Cambia a contenedor de diseño flexible */
    justify-content: space-between; /* Espacio uniforme entre los elementos internos */
    align-items: center; /* Centra verticalmente los elementos internos */
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1), 0 2px 4px rgba(0, 0, 0, 0.1); /* Sombras para el efecto 3D */
    margin-bottom: 3px; /* Espacio inferior para separar los mensajes */
    font-size: 13px;
    font-family: 'Segoe UI';
    font-weight: bold;
    position: relative; /* Agrega posición relativa al contenedor */
  }

  .pago-message.expanded {
    height: auto; /* Permite que la altura sea automática cuando se expande */
  }

  .pago-icon {
    width: 20px;
    height: 20px;
    margin-left: 10px; /* Ajusta el margen izquierdo según sea necesario */
  }

  .delete-button {
    color: red;
    border: none;
    padding: 1px;
    font-size: 12px;
    cursor: pointer;
    position: relative;
    top: 2px; /* Ajusta la posición superior según sea necesario */
  }

  .total-summary {
    text-align: left;
    margin-top: 350px;
    font-family: 'Segoe UI';
    font-size: 16px;
    font-weight: bold;
    margin-left: 180px;
    font-size: 20px;
    color: #ff0000;
    position: fixed;
    top: 180px; 
  }

  .propinas-container {
  margin-top: 320px;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 500px;
  margin-left: 700px;
  border-radius: 80%;
}

.propinas-button {
  background-color: white;
  color: black;
  border: 2px solid black;
  
  padding: 8px 16px;
  cursor: pointer;
  transition: background-color 0.3s, color 0.3s;
}

.propinas-button.completed {
  background-color: red;
  color: white;
  cursor: not-allowed; /* Desactiva el cursor cuando las propinas están completas */
}


  </style>
  