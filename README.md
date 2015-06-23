# Oka-proj
package marcox.oka;

import android.app.Activity;
import android.content.Context;
import android.database.Cursor;
import android.database.SQLException;
import android.database.sqlite.SQLiteDatabase;
import android.graphics.Color;
import android.os.Bundle;
import android.os.CountDownTimer;
import android.os.Handler;
import android.view.MotionEvent;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.TextView;

import java.io.IOException;
import java.sql.ResultSet;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;
import java.util.Timer;
import java.util.TimerTask;


public class PreguntaActivity extends Activity  implements View.OnTouchListener {

    private ImageView imgColorOka;
    private Button bt_resp1, bt_resp2, bt_resp3, bt_resp4;
    private TextView txt_pregunta, txt_jugador, tvTiempo;
    private Pregunta preg;
    private  Pregunta nuevaPreg;
    private int numMetodo;
    private static final int topePreguntas = 150;
    private AdminSQLiteOpenHelper admin;
    private String nombreJug, colorOka;
    private MyCount counter;
     PlayActivity play;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_pregunta);
        bt_resp1 = (Button) findViewById(R.id.bt_resp1);
        bt_resp2 = (Button) findViewById(R.id.bt_resp2);
        bt_resp3 = (Button) findViewById(R.id.bt_resp3);
        bt_resp4 = (Button) findViewById(R.id.bt_resp4);
        txt_jugador = (TextView) findViewById(R.id.txt_jugador);
        txt_pregunta = (TextView) findViewById(R.id.txt_pregunta);
        imgColorOka = (ImageView)findViewById(R.id.imgColorOka);
        tvTiempo = (TextView)findViewById(R.id.tvTiempo);
        bt_resp1.setOnTouchListener(this);
        bt_resp2.setOnTouchListener(this);
        bt_resp3.setOnTouchListener(this);
        bt_resp4.setOnTouchListener(this);

        counter = new MyCount(15000, 1000);
        counter.start();


        // map para poder cambiar el color de la oca del jugador que le toca
        final Map<String, Integer> map_oka1 = new HashMap<String, Integer>();
        map_oka1.put("mapaoka1", R.drawable.oca1_a);
        final Map<String, Integer> map_oka2 = new HashMap<String, Integer>();
        map_oka2.put("mapaoka2", R.drawable.oca2_a);
        final  Map<String, Integer> map_oka3 = new HashMap<String, Integer>();
        map_oka3.put("mapaoka3", R.drawable.oca3_a);
        final  Map<String, Integer> map_oka4 = new HashMap<String, Integer>();
        map_oka4.put("mapaoka4", R.drawable.oca4_a);
        final  Map<String, Integer> map_oka5 = new HashMap<String, Integer>();
        map_oka5.put("mapaoka5", R.drawable.oca5_a);
        final  Map<String, Integer> map_oka6 = new HashMap<String, Integer>();
        map_oka6.put("mapaoka6", R.drawable.oca6_a);


        Bundle bundle = getIntent().getExtras();
        nombreJug = bundle.getString("nombreJug");
        colorOka = bundle.getString("colorOka");


    play=new PlayActivity();
        admin = new AdminSQLiteOpenHelper(this);
        // ***** CARGAR PREGUNTA **////
        cargaPregunta();

        // Nombre del jugador
        txt_jugador.setText(nombreJug);
        //Color de la oka
        if (colorOka.equalsIgnoreCase("blanco")){
            imgColorOka.setImageResource(map_oka1.get("mapaoka1"));
        }else if (colorOka.equalsIgnoreCase("rosa")){
            imgColorOka.setImageResource(map_oka2.get("mapaoka2"));
        }else if (colorOka.equalsIgnoreCase("amarillo")){
            imgColorOka.setImageResource(map_oka3.get("mapaoka3"));
        }else if (colorOka.equalsIgnoreCase("verde")){
            imgColorOka.setImageResource(map_oka4.get("mapaoka4"));
        }else if (colorOka.equalsIgnoreCase("rojo")){
            imgColorOka.setImageResource(map_oka5.get("mapaoka5"));
        } else if (colorOka.equalsIgnoreCase("azul")){
            imgColorOka.setImageResource(map_oka6.get("mapaoka6"));
        }

        /*
        //TIEMPO PARA LA PREGUNTA
        Handler handler = new Handler();
        Runnable r=new Runnable() {
            @Override
            public void run() {
                finish();
            }
        };
        handler.postDelayed(r, 3000);
        */
    }

    public class MyCount extends CountDownTimer {

        public MyCount(long millisInFuture, long countDownInterval) {
            super(millisInFuture, countDownInterval);
        }

        @Override
        public void onFinish() {
            tvTiempo.setText("0");
            cierraPregunta();
        }

        @Override
        public void onTick(long millisUntilFinished) {
            tvTiempo.setText("" + millisUntilFinished / 1000);
            /*int a =Integer.parseInt( millisUntilFinished / 1000+"");
            if( a <= 3000){
                tvTiempo.setTextColor(Color.RED);
            }*/
        }
    }



    //CARGA LA PREGUNTA EN EL TEXTVIEW Y LAS RESPUESTAS EN SUS RESPECTIVOS BOTONES
    private void cargaPregunta() {

        if (buscarPregunta() != null) {
            preg = buscarPregunta();
        }


        txt_pregunta.setText(preg.getPreguntaPregunta());

        //Elegir aleatoriamente el metodo que cargará las respsuestas
        final Random rand = new Random();
        numMetodo = rand.nextInt(4) + 1; // del 1 al 4

        switch (numMetodo) {
            case 1:
                cargaMetodo1();
                break;
            case 2:
                cargaMetodo2();
                break;
            case 3:
                cargaMetodo3();
                break;
            case 4:
                cargaMetodo4();
                break;
        }
    }


    /// BUSCAR PREGUNTA
    private  Pregunta  buscarPregunta () {
        boolean encontrada = false;
        // while(encontrada=false){
        final Random rand = new Random();
        int idPreg = rand.nextInt(topePreguntas) + 1; // del 1 al tope de preguntas

        SQLiteDatabase db = admin.getWritableDatabase();
        Cursor c = db.rawQuery("SELECT _id, pregunta, respuesta_buena, respuesta_mala1, respuesta_mala2, respuesta_mala3, contestada FROM preguntasoka WHERE _id=" + idPreg, null);

        try {
                String preg, respuestaB, respuestaM1, respuestaM2, respuestaM3;
                boolean cont;
                if (c.moveToFirst()) {
                    // idPregu = Integer.parseInt(c.getString(0));
                    preg = c.getString(1);
                    respuestaB = c.getString(2);
                    respuestaM1 = c.getString(3);
                    respuestaM2 = c.getString(4);
                    respuestaM3 = c.getString(5);
                    String estaContestada = c.getString(6);

                    if (estaContestada.equalsIgnoreCase("true")) {
                        cont = true;
                        encontrada = false;
                    } else {
                        cont = false;
                    }
                        nuevaPreg = new Pregunta(idPreg, preg, respuestaB, respuestaM1, respuestaM2, respuestaM3, cont);
                        return nuevaPreg;
                    }


                }finally{
                    c.close();
                    db.close();
                }

             /*
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }*/return null;
    }

//** METODOS PARA CARGAR LAS PREGUNTAS **//
    private void cargaMetodo1() {
        bt_resp1.setText(preg.getRespuestaBuena());
        bt_resp2.setText(preg.getRespuestaMala1());
        bt_resp3.setText(preg.getRespuestaMala2());
        bt_resp4.setText(preg.getRespuestaMala3());
    }

    private void cargaMetodo2() {
        bt_resp1.setText(preg.getRespuestaMala1());
        bt_resp2.setText(preg.getRespuestaBuena());
        bt_resp3.setText(preg.getRespuestaMala2());
        bt_resp4.setText(preg.getRespuestaMala3());
    }

    private void cargaMetodo3() {
        bt_resp1.setText(preg.getRespuestaMala1());
        bt_resp2.setText(preg.getRespuestaMala2());
        bt_resp3.setText(preg.getRespuestaBuena());
        bt_resp4.setText(preg.getRespuestaMala3());
    }

    private void cargaMetodo4() {
        bt_resp1.setText(preg.getRespuestaMala3());
        bt_resp2.setText(preg.getRespuestaMala1());
        bt_resp3.setText(preg.getRespuestaMala2());
        bt_resp4.setText(preg.getRespuestaBuena());
    }

/*** METODO PARA MARCAR LA PREGUNTA RESPONDIDA  A TRUE**/
    private void marcaRespondida() {

    }

    /** METODO PARA SABER SI EL BOTON PULSADO CONTIENE LA RESPUESTA CORRECTA **/
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        switch (v.getId()) {
            case R.id.bt_resp1:
                if (numMetodo == 1) {
                    pintaCorrecto(bt_resp1);
                    acierto();

                }else{
                    pintaIncorrecto(bt_resp1);
                }
                break;
            case R.id.bt_resp2:
                if (numMetodo == 2) {
                    pintaCorrecto(bt_resp2);
                    acierto();

                }else{
                    pintaIncorrecto(bt_resp2);
                }
                break;
            case R.id.bt_resp3:
                if (numMetodo == 3) {
                   pintaCorrecto(bt_resp3);

                    acierto();
                }else{
                   pintaIncorrecto(bt_resp3);
                }
                break;
            case R.id.bt_resp4:
                if (numMetodo == 4) {
                    pintaCorrecto(bt_resp4);
                    acierto();
                }else{
                    pintaIncorrecto(bt_resp4);
                }
                break;
        }
        return false;
    }

    //**** METODOS  PARA PINTAR EN VERDE/ROJO RESPUESTAS CORRECTAS/INCORRECTAS**/
    public void pintaCorrecto(Button bt){
       bt.setBackgroundColor(Color.GREEN);
        desactivaBotones();
        cierraPregunta();
    }
    public void pintaIncorrecto(Button bt){
        bt.setBackgroundColor(Color.RED);
        desactivaBotones();
        cierraPregunta();
    }


    private void desactivaBotones(){
        bt_resp1.setOnTouchListener(null);
        bt_resp2.setOnTouchListener(null);
        bt_resp3.setOnTouchListener(null);
        bt_resp4.setOnTouchListener(null);
    }

    public void cierraPregunta() {
        final Timer t = new Timer();
        t.schedule(new TimerTask() {
            public void run() {
                counter.cancel();
                finish();
            }
        }, 2000);
    }
    
    public void acierto(){

         play.acerto();

     }


