// -------------------------------------- Constantes ------------------------------------------//

pins = [IN1 = D27, IN2 = D14, IN3 = D12, IN4 = D13];
botoes = [botAcionamento = D25, botMode = D34, FDCfechado = D15, FDCaberto = D19];
SCin_dig = D18;
D23.set();

//--------------------------------------- Variáveis -------------------------------------------//

var off = 0;
var on = 1;

var solto = 1;
var pressionado = 0;

var fechar = 0;
var abrir = 1;
var parar = 2;
var proximaAcao = parar;
var acaoAtual = fechar;

var fechada = 0;
var aberta = 1;
var entreaberta = 2;

var MotorA = 0;
var MotorB = 1;

var leituraSC = 0;
var chovendo = 0;
var semChuva = 1;

var autonomo = 0;
var dependente = 1;
var modo = autonomo;
D2.set();

var acionaPeloBot = 1;
var acionaPeloSC = 2;
var acionaPeloTMP = 3;
var acionamentoDependente = 4;
var operacao = acionaPeloSC;

var acionarMotorFlag = 0;
var horario = 1;
var antiHorario = 0;
var sentidoDefinido = horario;

var parado = 0;
var girando = 1;
var estadoMotor = parado;

//---------------------------------------- Inicializações -------------------------------------//

//Modo dos Pinos
pins.forEach(function(BOT){
      BOT.mode('output');
});

//Modo dos botões (fim de curso e acionador)
botoes.forEach(function(BOT){
  BOT.mode('input_pullup');
});

//Inicializacao Pins
IN1.reset();
IN2.reset();
IN3.reset();
IN4.reset();

//Led test resetado
D2.reset();

//Classe Motor
function MotorL298N(Motor) {
  this.Motor = Motor;
}

//Métodos (relacionados as funções desempenhadas pelo motor
MotorL298N.prototype.Girar = function(sentido){
    //Motor A selecionado
    if(this.Motor === 0){
      if(sentido === 0){
        //Gira o Motor A no sentido horario
        IN1.set();
        IN2.reset();
      }
      else if(sentido === 1){
        //Gira o Motor A no sentido anti-horario
        IN1.reset();
        IN2.set();
      }
    }
    //Motor B selecionado
    else if(this.Motor === 1){
      if(sentido === 0){
         //Gira o Motor B no sentido horario
         IN3.set();
         IN4.reset();
      }
      else if(sentido === 1){
        //Gira o Motor B no sentido anti-horario
        IN3.reset();
        IN4.set();
      }
   }
};

MotorL298N.prototype.Parar = function(sentido){
    //Motor A selecionado
    if(this.Motor === 0){
      IN1.set();
      IN2.set();
    }
    //Motor B selecionado
    else if(this.Motor === 1){
      IN3.set();
      IN4.set();
    }
};

//-------------------------------- Monitora o Sensor de chuva -----------------------------------//

//Modo do sensor de chuva
SCin_dig.mode('input_pullup');

function lerSC(){
  //A saida analogica do sensor serve para quantizar a chuva
  //A saida digital do sensor ficara em alto (led aceso) quando nao chover e baixo quando chover

  if(digitalRead(SCin_dig) === 1){
    console.log("Status clima: SEM CHUVA");
  }
  else{
    console.log("Status clima: CHOVENDO");
  }
  return digitalRead(SCin_dig);
}

//--------------------------------------- Monitora TMP100 ---------------------------------------//
I2C1.setup({ sda: D21, scl: D22, bitrate: 400000 });

//constantes

var slave_address = 0b1001001;

//Registradores do TMP100
var temperature_register = 0x00;

//Possiveis configurações de resolução
var set9bits  = 0x00;
var set10bits = 0x20;
var set11bits = 0x40;
var set12bits = 0x60;

//configura a resolução do TMP100
function TMP100SetResolution(slave_address, setBits){
  var configuration_register = 0x01;

  I2C1.writeTo(slave_address, [configuration_register, setBits]);
}

TMP100SetResolution(slave_address, set12bits);

function readTMP100(slave_address){
    var escrita_falsa = 0x00;
    var leitura = 0;
    var temp = 0;

//Escrita falsa no Slave Adress (TMP100)
	I2C1.writeTo(slave_address, escrita_falsa);
//Lê o Slave Adress (TMP100) e guarda a temperatura (em 2 bytes) na variavel leitura
    leitura = I2C1.readFrom(slave_address, 2);

//Conversão do valor lido para oC
	temp_inteira  = leitura[0];
	temp_fracionaria  = leitura[1]/256;
	temp = temp_inteira  + temp_fracionaria ;

//Printa o valor lido
	console.log('Temperatura: ' + temp + "ºC");
  return temp;
}

var temperaturaHigh = 26;  //oC
var temperaturaLow = 23;   //oC

//----------------------------------------- Uso do Bot ------------------------------------------//

//Acionamento da janela
setWatch(function(){
  if(modo === dependente){
    timerCounter = 0;
    console.log('ACIONAMENTO PELO BOT');
    operacao = acionamentoDependente;

    console.log('PRINT ESTADO MOTOR: '+ estadoMotor);

    if(estadoMotor === girando){
      proximaAcao = parar;
      console.log('AÇÃO DEFINIDA: PARAR');
    }
    else{//estadoMotor === parado
      if(acaoAtual === abrir){
        sentidoDefinido = antiHorario;
        proximaAcao = fechar;
        console.log('AÇÃO DEFINIDA: FECHAR');
      }
      else if(acaoAtual === fechar){
        sentidoDefinido = horario;
        proximaAcao = abrir;
        console.log('AÇÃO DEFINIDA: ABRIR');
      }
      else{//acaoAtual = parar;
        if(statusJanela() != fechada){
          proximaAcao = fechar;
        }
        else{
          proximaAcao = abrir;
        }
      }
    }
  }
}, botAcionamento, {repeat:true, edge:'falling', debounce:50});

//Alterar modo de funcionamento da janela
var timerCounter = 0;

setWatch(function(){
  if(estadoMotor != girando){
    timerCounter = 0;
    if(modo === dependente){
      console.log('JANELA ALTERADA PARA O MODO AUTÔNOMO');
      D2.set();
      modo = autonomo;
      operacao = acionaPeloSC;
    }
    else if(modo === autonomo){
      console.log('JANELA ALTERADA PARA O MODO DEPENDENTE');
      D2.reset();
      modo = dependente;
      operacao = acionamentoDependente;
    }
    if(statusJanela() === fechada){
      proximaAcao = abrir;
    }
    else if(statusJanela() === aberta){
      proximaAcao = fechar;
    }
  }
}, botMode, {repeat:true, edge:'falling', debounce:50});


//---------------------------- Interpretação do estado da janela --------------------------------//


//Interpreta se a janela está aberta ou fechada a partir dos botões fim de curso
function statusJanela(){
  var janela;

  if(digitalRead(FDCaberto) === pressionado && digitalRead(FDCfechado) === solto){
    janela = aberta;
    console.log('Status da janela: ABERTA');
  }
  else if(digitalRead(FDCfechado) === pressionado && digitalRead(FDCaberto) === solto){
    janela = fechada;
    console.log('Status da janela: FECHADA');
  }
  else{
    janela = entreaberta;
    console.log('Status da janela: ENTREABERTA');
  }
  return janela;
}

function defineSentido(){
  var sentido;
  var horario = 1;
  var antiHorario = 0;

  leitura = statusJanela();

  if(leitura != fechada){
    sentido = antiHorario;
  }
  else{
    sentido = horario;
  }
  return sentido;
}

//----------------------------------- Conexão via MQTT -------------------------------------------//

var mqtt = require("tinyMQTT").create('broker.hivemq.com');

mqtt.on('connected', function() {
  console.log('connected');
  mqtt.subscribe('ESP32-Janela/Acionar');
});

mqtt.on('message', function(msg) {
  var mensagem = JSON.parse(msg.message);
  if(mensagem === 'abrir'){
    console.log('ABRIR JANELA 1');
    modo = dependente;
    operacao = acionamentoDependente;
    sentidoDefinido = horario;
    proximaAcao = abrir;
  }
  else if(mensagem === 'fechar'){
    console.log('FECHAR JANELA 1');
    modo = dependente;
    operacao = acionamentoDependente;
    sentidoDefinido = antiHorario;
    proximaAcao = fechar;
  }
  timerCounter = 0;
});

var wifi = require('Wifi');
wifi.connect('janela', { password: '12345678' }, function() {
  console.log(`http://${wifi.getIP().ip}`);

  mqtt.connect();
});

//-------------------------------------- Acionamento ---------------------------------------------//
//inicializa o motor B do shield (L298N)
var motor = new MotorL298N(MotorB);

function acionamento(acionarMotorFlag, sentidoDefinido){
  if(acionarMotorFlag === on){
    motor.Girar(sentidoDefinido);
  }
  else{
    motor.Parar();
  }
}

var timeLoop = 1000;
var timeUp = 45;

//---------------------------------------- Estados ------------------------------------------------//

setInterval(function(){
  //timeWatch
  console.log('Tempo: ' + timerCounter + 's');
  if(modo === dependente){
    timerCounter = timerCounter + 1;
    if(timerCounter === timeUp){
      if(estadoMotor === parado){
        timerCounter = 0;
        modo = autonomo;
        operacao = acionaPeloSC;
        D2.set();
        console.log('ALTERADA AUTOMATICAMENTE PARA O MODO AUTÔNOMO');
      }
    }
  }

  switch(modo){
    case autonomo:
      sentidoDefinido = defineSentido();
      switch(operacao){
        case acionaPeloSC:
          if(acionarMotorFlag === off || estadoMotor === girando){
            console.log('ESTADO ACIONA PELO SC');
            leitura = lerSC();
            if(leitura === chovendo && statusJanela() != fechada){
              acionarMotorFlag = on;
              estadoMotor = girando;
              acionamento(acionarMotorFlag, sentidoDefinido);
            }
            else{
              acionarMotorFlag = off;
              estadoMotor = parado;
              acionamento(acionarMotorFlag, sentidoDefinido);
            }
          }
          if(leitura === chovendo){
            operacao = acionaPeloSC;
          }
          else{
            operacao = acionaPeloTMP;
          }
        break;

        case acionaPeloTMP:
          console.log('ESTADO ACIONA PELO TMP');
          var leituraTemp = readTMP100(slave_address);
          //Se a temperatura passar de 25oC, abre a janela
          if(leituraTemp >= temperaturaHigh || estadoMotor === girando){
            estadoMotor = girando;
            acionarMotorFlag = on;
            acionamento(acionarMotorFlag, horario);
            if(statusJanela() === aberta){
              estadoMotor = parado;
              acionarMotorFlag = off;
              acionamento(acionarMotorFlag, horario);
              operacao = acionaPeloSC;
            }
          }
          else if (leituraTemp <= temperaturaLow || estadoMotor === girando){
            estadoMotor = girando;
            acionarMotorFlag = on;
            acionamento(acionarMotorFlag, antiHorario);
            if(statusJanela() === fechada){
              estadoMotor = parado;
              acionarMotorFlag = off;
              acionamento(acionarMotorFlag, horario);
              operacao = acionaPeloSC;
            }
          }
          if(estadoMotor === girando){
            operacao = acionaPeloTMP;
          }
          else if(estadoMotor === parado){
            operacao = acionaPeloSC;
          }
        break;
      }
    break;

    case dependente:
      switch(operacao){
        case acionamentoDependente:
          switch(proximaAcao){
            case fechar:
              console.log('FECHAR JANELA 2');
              acionarMotorFlag = on;
              acionamento(acionarMotorFlag, sentidoDefinido);
              acaoAtual = fechar;
              estadoMotor = girando;
              if(statusJanela() === fechada){
                proximaAcao = parar;
                acionarMotorFlag = off;
                estadoMotor = parado;
                acionamento(acionarMotorFlag, sentidoDefinido);
              }
            break;
            case abrir:
              console.log('ABRIR JANELA 2');
              acionarMotorFlag = on;
              acionamento(acionarMotorFlag, sentidoDefinido);
              acaoAtual = abrir;
              estadoMotor = girando;
              if(statusJanela() === aberta){
                proximaAcao = parar;
                acionarMotorFlag = off;
                estadoMotor = parado;
                acionamento(acionarMotorFlag, sentidoDefinido);
              }
            break;
            case parar:
              acionarMotorFlag = off;
              acionamento(acionarMotorFlag, sentidoDefinido);
              estadoMotor = parado;
            break;
          }
        break;
      }
    break;
  }
  console.log('                                                                              ');
}, timeLoop);
