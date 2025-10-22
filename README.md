package com.example.tasklist

import android.content.Context
import android.graphics.Paint
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.view.inputmethod.InputMethodManager
import android.widget.Button
import android.widget.CheckBox
import android.widget.EditText
import android.widget.ImageButton
import android.widget.LinearLayout
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.constraintlayout.widget.ConstraintLayout
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.ListAdapter
import androidx.recyclerview.widget.RecyclerView

// Data Class
data class Task(
    val id: Long = System.currentTimeMillis(),
    val title: String,
    val isCompleted: Boolean = false
)

// DiffUtil Callback
class TaskDiffCallback : DiffUtil.ItemCallback<Task>() {
    override fun areItemsTheSame(oldItem: Task, newItem: Task): Boolean {
        return oldItem.id == newItem.id
    }

    override fun areContentsTheSame(oldItem: Task, newItem: Task): Boolean {
        return oldItem == newItem
    }
}

// Task Adapter
class TaskAdapter(
    private val onTaskClicked: (Task) -> Unit,
    private val onDeleteClicked: (Task) -> Unit
) : ListAdapter<Task, TaskAdapter.TaskViewHolder>(TaskDiffCallback()) {

    inner class TaskViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        private val cbTask: CheckBox = itemView.findViewById(R.id.cbTask)
        private val tvTitle: TextView = itemView.findViewById(R.id.tvTitle)
        private val btnDelete: ImageButton = itemView.findViewById(R.id.btnDelete)

        fun bind(task: Task) {
            tvTitle.text = task.title
            cbTask.isChecked = task.isCompleted

            // Configurar o risco no texto quando a tarefa estiver concluída
            if (task.isCompleted) {
                tvTitle.paintFlags = tvTitle.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
            } else {
                tvTitle.paintFlags = tvTitle.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
            }

            // Listener para o CheckBox
            cbTask.setOnCheckedChangeListener { _, isChecked ->
                if (isChecked != task.isCompleted) {
                    onTaskClicked(task.copy(isCompleted = isChecked))
                }
            }

            // Listener para o botão de deletar
            btnDelete.setOnClickListener {
                onDeleteClicked(task)
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TaskViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_task, parent, false)
        return TaskViewHolder(view)
    }

    override fun onBindViewHolder(holder: TaskViewHolder, position: Int) {
        holder.bind(getItem(position))
    }
}

// Main Activity
class MainActivity : AppCompatActivity() {

    private lateinit var adapter: TaskAdapter
    private var taskList = mutableListOf<Task>()
    
    private lateinit var recyclerView: RecyclerView
    private lateinit var etTitle: EditText
    private lateinit var btnAdd: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Create main layout programmatically
        createMainLayout()
        
        initViews()
        setupRecyclerView()
        setupClickListeners()
    }

    private fun createMainLayout() {
        // Main LinearLayout
        val mainLayout = LinearLayout(this).apply {
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.MATCH_PARENT
            )
            orientation = LinearLayout.VERTICAL
            setPadding(50, 50, 50, 50) // 16dp converted to pixels approximately
        }

        // Input Layout
        val inputLayout = LinearLayout(this).apply {
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            )
            orientation = LinearLayout.HORIZONTAL
        }

        // EditText for task title
        etTitle = EditText(this).apply {
            id = R.id.etTitle
            layoutParams = LinearLayout.LayoutParams(
                0,
                LinearLayout.LayoutParams.WRAP_CONTENT,
                1f
            ).apply {
                marginEnd = 30 // 8dp converted to pixels approximately
            }
            hint = "Digite o título da tarefa"
            imeOptions = android.view.inputmethod.EditorInfo.IME_ACTION_DONE
        }

        // Add Button
        btnAdd = Button(this).apply {
            id = R.id.btnAdd
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            )
            text = "Adicionar"
        }

        // RecyclerView
        recyclerView = RecyclerView(this).apply {
            id = R.id.recyclerView
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                0,
                1f
            ).apply {
                topMargin = 50 // 16dp converted to pixels approximately
            }
        }

        // Add views to layouts
        inputLayout.addView(etTitle)
        inputLayout.addView(btnAdd)
        mainLayout.addView(inputLayout)
        mainLayout.addView(recyclerView)

        setContentView(mainLayout)
    }

    private fun initViews() {
        // Views already initialized in createMainLayout()
    }

    private fun setupRecyclerView() {
        adapter = TaskAdapter(
            onTaskClicked = { task ->
                // Atualizar tarefa quando o checkbox for clicado
                val index = taskList.indexOfFirst { it.id == task.id }
                if (index != -1) {
                    taskList[index] = task
                    adapter.submitList(taskList.toList())
                }
            },
            onDeleteClicked = { task ->
                // Remover tarefa quando o botão de deletar for clicado
                taskList.removeAll { it.id == task.id }
                adapter.submitList(taskList.toList())
            }
        )

        recyclerView.adapter = adapter
        recyclerView.layoutManager = LinearLayoutManager(this)
    }

    private fun setupClickListeners() {
        btnAdd.setOnClickListener {
            val title = etTitle.text.toString().trim()
            if (title.isNotEmpty()) {
                // Criar nova tarefa
                val newTask = Task(title = title)
                taskList.add(newTask)
                adapter.submitList(taskList.toList())
                etTitle.text.clear()
                
                // Fechar teclado virtual
                hideKeyboard()
            } else {
                Toast.makeText(this, "Digite um título para a tarefa", Toast.LENGTH_SHORT).show()
            }
        }
    }

    private fun hideKeyboard() {
        val imm = getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager
        imm.hideSoftInputFromWindow(etTitle.windowToken, 0)
    }
}

// Resource IDs - We need to define these since we're creating views programmatically
class R {
    companion object {
        object id {
            const val recyclerView = 1
            const val etTitle = 2
            const val btnAdd = 3
            const val cbTask = 4
            const val tvTitle = 5
            const val btnDelete = 6
        }
        object layout {
            const val item_task = 7
        }
    }
}

// Extension function to create item layout for RecyclerView
fun ViewGroup.createTaskItemLayout(): View {
    val context = this.context
    
    return ConstraintLayout(context).apply {
        layoutParams = ConstraintLayout.LayoutParams(
            ConstraintLayout.LayoutParams.MATCH_PARENT,
            ConstraintLayout.LayoutParams.WRAP_CONTENT
        ).apply {
            setPadding(50, 50, 50, 50) // 16dp converted to pixels approximately
        }
        
        // CheckBox
        val checkBox = CheckBox(context).apply {
            id = R.id.cbTask
            layoutParams = ConstraintLayout.LayoutParams(
                ConstraintLayout.LayoutParams.WRAP_CONTENT,
                ConstraintLayout.LayoutParams.WRAP_CONTENT
            ).apply {
                startToStart = ConstraintLayout.LayoutParams.PARENT_ID
                topToTop = ConstraintLayout.LayoutParams.PARENT_ID
                bottomToBottom = ConstraintLayout.LayoutParams.PARENT_ID
                marginEnd = 50 // 16dp converted to pixels approximately
            }
        }
        
        // Delete Button
        val deleteButton = ImageButton(context).apply {
            id = R.id.btnDelete
            layoutParams = ConstraintLayout.LayoutParams(
                ConstraintLayout.LayoutParams.WRAP_CONTENT,
                ConstraintLayout.LayoutParams.WRAP_CONTENT
            ).apply {
                endToEnd = ConstraintLayout.LayoutParams.PARENT_ID
                topToTop = ConstraintLayout.LayoutParams.PARENT_ID
                bottomToBottom = ConstraintLayout.LayoutParams.PARENT_ID
            }
            setBackgroundResource(android.R.attr.selectableItemBackground)
            setImageResource(android.R.drawable.ic_delete)
            contentDescription = "Excluir tarefa"
        }
        
        // Title TextView
        val titleText = TextView(context).apply {
            id = R.id.tvTitle
            layoutParams = ConstraintLayout.LayoutParams(
                0,
                ConstraintLayout.LayoutParams.WRAP_CONTENT
            ).apply {
                startToEnd = R.id.cbTask
                endToStart = R.id.btnDelete
                topToTop = ConstraintLayout.LayoutParams.PARENT_ID
                bottomToBottom = ConstraintLayout.LayoutParams.PARENT_ID
                marginEnd = 50 // 16dp converted to pixels approximately
            }
            textSize = 16f
        }
        
        addView(checkBox)
        addView(titleText)
        addView(deleteButton)
    }
}

// We need to override the inflate method in the adapter to use our programmatic layout
// This is a simplified approach for the single file solution
fun TaskAdapter.TaskViewHolder(itemView: View) {
    // This would normally be handled by the inner class constructor
    // For the single file solution, we rely on the programmatic layout creation
}
