Android-Restaurant
==================
package com.example.restaurant;

import android.app.TabActivity;
import android.content.Context;
import android.database.Cursor;
import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.view.LayoutInflater;
import android.widget.AdapterView;
import android.widget.CursorAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageView;
import android.widget.ListView;
import android.widget.RadioGroup;
import android.widget.TabHost;
import android.widget.TextView;

public class MainActivity extends TabActivity {
	Cursor model = null;
	RestautantAdapter adapter = null;
	EditText name = null;
	EditText address = null;
	EditText notes = null;
	RadioGroup types = null;
	RestautantHelper helper = null;

	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		helper = new RestautantHelper(this);

		name = (EditText) findViewById(R.id.name);
		address = (EditText) findViewById(R.id.addr);
		notes = (EditText) findViewById(R.id.notes);
		types = (RadioGroup) findViewById(R.id.types);
		Button save = (Button) findViewById(R.id.save);
		save.setOnClickListener(onSave);
		ListView list = (ListView) findViewById(R.id.restaurants);
		model = helper.getAll();
		startManagingCursor(model);
		adapter = new RestautantAdapter(model);
		list.setAdapter(adapter);

		TabHost.TabSpec spec = getTabHost().newTabSpec("tag1");

		spec.setContent(R.id.restaurants);
		spec.setIndicator("List", getResources().getDrawable(R.drawable.list));

		getTabHost().addTab(spec);

		spec = getTabHost().newTabSpec("tag2");
		spec.setContent(R.id.details);
		spec.setIndicator("Details", getResources()
				.getDrawable(R.drawable.list));
		getTabHost().addTab(spec);
		getTabHost().setCurrentTab(0);

		list.setOnItemClickListener(onListClick);
	}

	@Override
	public void onDestroy() {
		super.onDestroy();

		helper.close();
	}

	private View.OnClickListener onSave = new View.OnClickListener() {
		public void onClick(View v) {

			String type = null;
			switch (types.getCheckedRadioButtonId()) {
			case R.id.sit_down:

				type = "sit_down";
				break;
			case R.id.take_out:
				type = "take_out";
				break;
			case R.id.delivery:
				type = "delivery";
				break;
			}
			helper.insert(name.getText().toString(), address.getText()
					.toString(), type, notes.getText().toString());
			model.requery();
		}
	};
	private AdapterView.OnItemClickListener onListClick = new AdapterView.OnItemClickListener() {
		public void onItemClick(AdapterView<?> parent, View view, int position,
				long id) {
			model.moveToPosition(position);

			name.setText(helper.getName(model));
			address.setText(helper.getAddress(model));
			notes.setText(helper.getNotes(model));
			if (helper.getType(model).equals("sit_down")) {
				types.check(R.id.sit_down);
			}

			else if (helper.getType(model).equals("take_out")) {
				types.check(R.id.take_out);
			} else {
				types.check(R.id.delivery);
			}

			getTabHost().setCurrentTab(1);
		}
	};

	class RestautantAdapter extends CursorAdapter {
		RestautantAdapter(Cursor c) {

			super(MainActivity.this, c);
		}

		@Override
		public void bindView(View row, Context ctxt, Cursor c) {
			RestautantHolder holder = (RestautantHolder) row.getTag();
			holder.populateFrom(c, helper);
		}

		@Override
		public View newView(Context ctxt, Cursor c, ViewGroup parent) {
			LayoutInflater inflater = getLayoutInflater();
			View row = inflater.inflate(R.layout.row, parent, false);
			RestautantHolder holder = new RestautantHolder(row);

			row.setTag(holder);
			return (row);
		}
	}

	static class RestautantHolder {
		private TextView name = null;
		private TextView address = null;
		private ImageView icon = null;
		private View row = null;

		RestautantHolder(View row) {
			this.row = row;

			name = (TextView) row.findViewById(R.id.title);
			address = (TextView) row.findViewById(R.id.address);
			icon = (ImageView) row.findViewById(R.id.icon);
		}

		void populateFrom(Cursor c, RestautantHelper helper) {
			name.setText(helper.getName(c));
			address.setText(helper.getAddress(c));

			if (helper.getType(c).equals("sit_down")) {
				icon.setImageResource(R.drawable.two);
			} else if (helper.getType(c).equals("take_out")) {
				icon.setImageResource(R.drawable.three);
			} else {
				icon.setImageResource(R.drawable.four);
			}
		}
	}
}
