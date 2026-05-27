📘 TP Android – ViewModel + LiveData (Java)

🧩 Étape 1 : Création du projet (5 min)

* Ouvrir Android Studio
* New Project → Empty Activity
* Nom : ViewModelLiveDataDemoEnrichi
* Language : Java
* Minimum SDK : API 24

⸻

📦 Ajout des dépendances (IMPORTANT)

Dans build.gradle (Module: app) :

def lifecycle_version = "2.10.0"
implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version"
implementation "androidx.lifecycle:lifecycle-livedata:$lifecycle_version"

📌 Puis :

* Sync Now

🧠 Explication

* ViewModel = conservation des données lors des rotations
* LiveData = mise à jour automatique de l’UI

⸻

🧪 Partie 1 : Version SANS ViewModel (problème classique)

📱 Layout : activity_main.xml

<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:gravity="center"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/tvCount"
        android:text="0"
        android:textSize="80sp"
        android:layout_marginBottom="32dp"/>
    <Button
        android:id="@+id/btnIncrement"
        android:text="INCRÉMENTER"/>
    <Button
        android:id="@+id/btnDecrement"
        android:text="DÉCRÉMENTER"/>
    <Button
        android:id="@+id/btnReset"
        android:text="RÉINITIALISER"/>
</LinearLayout>

⸻

📱 MainActivity (sans ViewModel)

public class MainActivity extends AppCompatActivity {
    private int count = 0;
    private TextView tvCount;
    private Button btnIncrement, btnDecrement, btnReset;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tvCount = findViewById(R.id.tvCount);
        btnIncrement = findViewById(R.id.btnIncrement);
        btnDecrement = findViewById(R.id.btnDecrement);
        btnReset = findViewById(R.id.btnReset);
        if (savedInstanceState != null) {
            count = savedInstanceState.getInt("count_key", 0);
        }
        updateUI();
        btnIncrement.setOnClickListener(v -> { count++; updateUI(); });
        btnDecrement.setOnClickListener(v -> { count--; updateUI(); });
        btnReset.setOnClickListener(v -> { count = 0; updateUI(); });
    }
    private void updateUI() {
        tvCount.setText(String.valueOf(count));
    }
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putInt("count_key", count);
    }
}

⚠️ Limite

* Perte de données lors de scénarios complexes
* Gestion manuelle fragile

⸻

🚀 Partie 2 : Version AVEC ViewModel + LiveData

⸻

🧠 ViewModel : CounterViewModel.java

public class CounterViewModel extends ViewModel {
    private final MutableLiveData<Integer> countLiveData = new MutableLiveData<>();
    public CounterViewModel() {
        countLiveData.setValue(0);
    }
    public void increment() {
        Integer current = countLiveData.getValue();
        if (current != null) countLiveData.setValue(current + 1);
    }
    public void decrement() {
        Integer current = countLiveData.getValue();
        if (current != null) countLiveData.setValue(current - 1);
    }
    public void reset() {
        countLiveData.setValue(0);
    }
    public LiveData<Integer> getCount() {
        return countLiveData;
    }
}

⸻

📱 MainActivity (avec ViewModel)

public class MainActivity extends AppCompatActivity {
    private CounterViewModel viewModel;
    private TextView tvCount;
    private Button btnIncrement, btnDecrement, btnReset;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tvCount = findViewById(R.id.tvCount);
        btnIncrement = findViewById(R.id.btnIncrement);
        btnDecrement = findViewById(R.id.btnDecrement);
        btnReset = findViewById(R.id.btnReset);
        viewModel = new ViewModelProvider(this).get(CounterViewModel.class);
        viewModel.getCount().observe(this, new Observer<Integer>() {
            @Override
            public void onChanged(Integer value) {
                tvCount.setText(String.valueOf(value));
            }
        });
        btnIncrement.setOnClickListener(v -> viewModel.increment());
        btnDecrement.setOnClickListener(v -> viewModel.decrement());
        btnReset.setOnClickListener(v -> viewModel.reset());
    }
}

⸻

🧠 Explication importante

🟢 ViewModel

* Survit aux rotations
* Sépare logique / UI
* Évite onSaveInstanceState

🟢 LiveData

* Met à jour automatiquement l’UI
* Respecte le cycle de vie
* Évite les memory leaks

⸻

🧪 Tests à faire

✔️ Test 1

* Incrémenter 10 fois
* Rotation écran
* Résultat : valeur conservée

✔️ Test 2

* Changer thème
* Résultat : UI reste cohérente

✔️ Test 3

adb shell am kill com.example.viewmodellivedatademoenrichi

* Redémarrage → données toujours gérées proprement


